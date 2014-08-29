
#####实时推荐

[http://pkghosh.wordpress.com/2014/04/14/making-recommendations-in-real-time/](http://pkghosh.wordpress.com/2014/04/14/making-recommendations-in-real-time/)

根据用户在较短的一个时间窗口内的行为做推荐，实时特征结合协同过滤推荐
######Hadoop批处理计算item相关矩阵

storm用来处理用户的实时活动流，找到推荐item，hadoop用来从历史数据中计算相关矩阵，redis作为各种大数据的子系统

user engagement event stream：
* userId
* sessionId
* itemId
* eventType

最终的相关矩阵存储在redis缓存，然后被storm消费，其中itemId为cache key,list为相关itemid,相关系数。随着顾客群的变化，货品item的变化以及用户行为的变化，这个矩阵要周期性的进行计算。(模型漂移)
重新计算的频率以及时间窗口的长度应该由环境的变化动态程度来决定，例如一周更新一次，使用之前6个月的数据。

######Storm实时推荐

基于item相关矩阵的协同过滤需要两种输入：
* 用户打分过的item和分数
* item相关矩阵

第二个item相关矩阵由历史数据hadoop计算得到,第一个storm会用用户实时活动流来估计用户和其他item的相关度

元json数据在storm启动之前装载进redis缓存，在bolt初始化的时候由缓存读入并反序列化为一个java对象

storm topology包括redis队列的spout和负责处理的bolt，spout和bolt的并行度由参数来配置，事件tuples是通过userId来分片的，所以同一个用户的行为始终被同一个bolt处理

bolt在内存中维护一个(userId,itemId)对的事件queue，事件窗口的过期设定规则如下：
* session：每当一个新seesion开始，queue被清空
* 时间窗口：只有最近一段时间窗口的事件被保留，旧的抛弃
* 事件计数：只保留一定数量的事件

时间窗口取决于近期行为对推荐结果的影响，短则最近的浏览数据决定推荐结果

item相似度矩阵的缓存分两层，第一层基于google guava cache
,当第一层cache miss的时候查找redis cache

推荐结果输出可以写入redis queue或缓存.一般来说用户每一个行为都产生一个新的推荐集合，如果客户端不需要读取所有推荐可以使用缓存,Redis Cache中userId为key,cache则返回最近的推荐列表。

#####实时处理步骤

* redis spout把事件tuple传给相应userId的bolt
* Bolt把事件对象基于userId,itemId写入内存队列，根据过期规则，剔除事件对象
* 对每个事件队列，扫描所有事件选出用户有最高意愿的事件，同时计数事件.事件的计数和类型用来计算item的rating
* 从缓存查找每个item对应的相关向量，使用estimated rating和相关向量，计算相关item的相似度
* 对所有item计算得到的相关向量聚合并求均值
* 聚合并求均值的向量按rating降序排列并取top k为推荐
* top k推荐写入redis队列或缓存，userId为缓存key

#####输入输出样本

实时事件流
	
	userID, sessionID, itemID and event.
	
	IRJKZ9226AHZ, 506bb5e6-c264-11e3-9c30-1c659d492701, CY7TJDN8O8, 2
	R29PBQ1757I1, 506bc482-c264-11e3-b647-1c659d492701, A501JBCS5D, 3
	R29PBQ1757I1, 506bc482-c264-11e3-b647-1c659d492701, M1BSEJZVQL, 0

推荐输出样本：

	userID和top 5 item及分数
	
	IBNZZA2ZN4EQ,FC3R48KUMX:19440,3EJ3IH3260:18920,K3VYYXRRZ7:15040,JXY0NCS22Y:11840,VSO7PV3GU4:3440
	ZL1EDYT4RX0A,A3S51X0MXQ:3600,FKV3M5HUPK:3208,SV24KIASSU:2912,ISWIQDPZGT:2896,YO3XIIL7ES:472
	IN2F94UKNLR3,I871XXOSYF:15986,S910V5YJF6:14200,CY7TJDN8O8:12695,1H092ZJIST:11720,Y2NBXWIC5S:9726
	ZDJ44TJ21AYM,O9LKR00D2U:8280,2KYFI6J3CD:7820,48X0F01WR2:7100,YLT6QBJ433:7080,TNL7373JAE:1800
	IN2F94UKNLR3,CY7TJDN8O8:16228,I871XXOSYF:15986,1H092ZJIST:14944,Y2NBXWIC5S:14690,S910V5YJF6:14200

#####Storm的局限

不支持状态，在本案例中状态包括：
* 内存中的事件队列
* 缓存的item相关向量
* 缓存事件 Cached event to rating mapping meta data

节点挂了，内存中的事件队列就失去了，然后一个新的bolt启动作为failure recover，bolt对相应的用户的推荐数量会很少，因为内存中的队列没有被充分处理，最终稳定，这种恢复叫