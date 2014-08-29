#####MyISAM InnoDB 区别

http://www.cnblogs.com/youxin/p/3359132.html

#####一些常用Mysql命令


* 查看表engine

	SHOW TABLE STATUS WHERE Name = 'tablename'

* 创建索引 

在执行CREATE TABLE语句时可以创建索引，也可以单独用CREATE INDEX或ALTER TABLE来为表增加索引。

1．ALTER TABLE
ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引。

	ALTER TABLE table_name ADD INDEX index_name (column_list)
	ALTER TABLE table_name ADD UNIQUE (column_list)
	ALTER TABLE table_name ADD PRIMARY KEY (column_list)


其中table_name是要增加索引的表名，column_list指出对哪些列进行索引，多列时各列之间用逗号分隔。索引名index_name可选，缺省时，MySQL将根据第一个索引列赋一个名称。另外，ALTER TABLE允许在单个语句中更改多个表，因此可以在同时创建多个索引。

2．CREATE INDEX

CREATE INDEX可对表增加普通索引或UNIQUE索引。
	
	CREATE INDEX index_name ON table_name (column_list)
	CREATE UNIQUE INDEX index_name ON table_name (column_list)

table_name、index_name和column_list具有与ALTER TABLE语句中相同的含义，索引名不可选。另外，不能用CREATE INDEX语句创建PRIMARY KEY索引。


#####NoSQL对比

[cassandra-vs-mongodb-vs-couchdb-vs-redis](ttp://kkovacs.eu/cassandra-vs-mongodb-vs-couchdb-vs-edis)

[NOSQL非关系型数据库学习（四）这样对比下HBASE, MEMCACHED, MONGODB, REDIS和SOLR](http://www.blogjava.net/crazycy/archive/2014/01/14/408880.html)

[Hbase快速开始shell](http://www.cnblogs.com/kaituorensheng/p/3814925.html)

#####Redis适用场景

* 取最新N个数据的操作
* top N 排行榜
* 需要精确设定过期时间
* 计数器
* unique操作
* 获取某段时间所有数据排重值
* 实时系统，反垃圾系统
* pub，su（发布、订阅）构建实时消息系统
* 构建队列系统
* 缓存
* 
#####Redis的11个应用案例

1．在主页中显示最新的项目列表。

Redis使用的是常驻内存的缓存，速度非常快。LPUSH用来插入一个内容ID，作为关键字存储在列表头部。LTRIM用来限制列表中的项目数最多为5000。如果用户需要的检索的数据量超越这个缓存容量，这时才需要把请求发送到数据库。

2．删除和过滤。

如果一篇文章被删除，可以使用LREM从缓存中彻底清除掉。 

3．排行榜及相关问题。

排行榜（leader board）按照得分进行排序。ZADD命令可以直接实现这个功能，而ZREVRANGE命令可以用来按照得分来获取前100名的用户，ZRANK可以用来获取用户排名，非常直接而且操作容易。

4．按照用户投票和时间排序。

这就像Reddit的排行榜，得分会随着时间变化。LPUSH和LTRIM命令结合运用，把文章添加到一个列表中。一项后台任务用来获取列表，并重新计算列表的排序，ZADD命令用来按照新的顺序填充生成列表。列表可以实现非常快速的检索，即使是负载很重的站点。

5．过期项目处理。

使用unix时间作为关键字，用来保持列表能够按时间排序。对current_time和time_to_live进行检索，完成查找过期项目的艰巨任务。另一项后台任务使用ZRANGE...WITHSCORES进行查询，删除过期的条目。

6．计数。

进行各种数据统计的用途是非常广泛的，比如想知道什么时候封锁一个IP地址。INCRBY命令让这些变得很容易，通过原子递增保持计数；GETSET用来重置计数器；过期属性用来确认一个关键字什么时候应该删除。

7．特定时间内的特定项目。

这是特定访问者的问题，可以通过给每次页面浏览使用SADD命令来解决。SADD不会将已经存在的成员添加到一个集合。

8．实时分析正在发生的情况，用于数据统计与防止垃圾邮件等。

使用Redis原语命令，更容易实施垃圾邮件过滤系统或其他实时跟踪系统。

9．Pub/Sub。

在更新中保持用户对数据的映射是系统中的一个普遍任务。Redis的pub/sub功能使用了SUBSCRIBE、UNSUBSCRIBE和PUBLISH命令，让这个变得更加容易。 

10．队列。

在当前的编程中队列随处可见。除了push和pop类型的命令之外，Redis还有阻塞队列的命令，能够让一个程序在执行时被另一个程序添加到队列。你也可以做些更有趣的事情，比如一个旋转更新的RSS feed队列。

11．缓存。

Redis缓存使用的方式与memcache相同。

网络应用不能无休止地进行模型的战争，看看这些Redis的原语命令，尽管简单但功能强大，把它们加以组合，所能完成的就更无法想象。当然，你可以专门编写代码来完成所有这些操作，但Redis实现起来显然更为轻松。

|：--：|：--：|：————：|
|redis|mysql|mongodb|
|库|库|库|

#####Redis vs. Memcached

1、存储方式

Memecache把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小。
Redis有部份存在硬盘上，这样能保证数据的持久性。
2、数据支持类型

Memcache对数据类型支持相对简单。
Redis有复杂的数据类型。
3、使用底层模型不同

它们之间底层实现方式 以及与客户端之间通信的应用协议不一样。
Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。
