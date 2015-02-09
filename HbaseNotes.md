手动安装Hbase

硬件资源规划

#####配置文件

######分布式配置

1. 修改hbase-site.xml配置文件，在configuration里添加
	
	<property>
	    <name>hbase.rootdir</name>
	    <value>hdfs://zktest1:9100/hbase</value>
	</property>
	<property>
	    <name>hbase.cluster.distributed</name>
	    <value>true</value>
	</property>
	<property>
	      <name>hbase.zookeeper.quorum</name>
	      <value>zktest1,zktest2,zktest3</value>
	</property>
    <property>
	      <name>hbase.zookeeper.property.dataDir</name>
	      <value>/home/vagrant/data/zookeeper</value>
	</property>
	<property>
	    <name>hbase.master.info.bindAddress</name>
	    <value>zktest1</value>
	<description>The bind address for the HBase Master web UI
	</description>
	</property>

2. 修改hbase-env.sh文件，加入

		export JAVA_HOME=/home/vagrant/software/jdk1.7.0_60
		export HBASE_MANAGES_ZK=true

3. 编辑regionservers文件，添加两个RegionServer：
	
	zktest2
	zktest3

4. 配置另外两台

将hbase安装文件拷贝到另两台机器：

	scp -r hbase-0.94.15-cdh4.7.0 vagrant@zktest2:/home/vagrant/software
	scp -r hbase-0.94.15-cdh4.7.0 vagrant@zktest3:/home/vagrant/software



######伪分布式配置

hbase-site.xml

	<configuration>
	<property>
	  <name>hbase.cluster.distributed</name>
	  <value>true</value>
	</property>
	<property>
	  <name>hbase.master.wait.on.regionservers.mintostart</name>
	  <value>1</value>
	</property>
	<property>
	  <name>hbase.rootdir</name>
	  <value>hdfs://finger-test1:9100</value>
	</property>
	</configuration>

启动

	bin/start-hbase.sh


#####运行


#####应用场景

多版本数据 
如上文提到的根据Row key和Column key定位到的Value可以有任意数量的版本值，因此对于需要存储变动历史记录的数据，用HBase就非常方便了。比如上例中的author的Address是会变动的，业务上一般只需要最新的值，但有时可能需要查询到历史值。

#####什么情况使用Hbase?

1.成熟的数据分析主题，查询模式已经确定并且不易改变

2.传统的关系型数据库已经无法承受负荷，**高速插入，大量读取**

3.适合海量的，但是同时也是简单的操作，比如说key-value的操作

***********************************************************

关系型数据库RDBMS所遇到的困难？

1.简单的事情只要上了高负载高并发就会变成很复杂的事情

2.Order by需要耗费很多的性能

3.高负载大量的数据量的发生，但是又无法进行分布式处理。

4.传统的RDBMS无法进行实时处理。如果顾客需要看到自己的访问记录的足迹，并不能通过缓存机制实时的进行统计出来，所以需要实时的文件处理

***********************************************************

Hbase的优势？

1.hbase就是 天生面向**时间戳**进行查询的

2.基于行键做B+树 ，查询异常快速，特别是最近的数据是放在memory store（几乎都是放在内存之中），减少了I/O的操作。

3.利用分布式化解负荷

***********************************************************
提高hbase集群性能

hbased的分布式原理是通过rowkey的范围进行分布式的

****************************************************************
应用场景： 看过本商品的用户还浏览了xx商品

P-U 行键是productid 列族和列为user:userid
U-P 行键是userid    列族和列为product：productid


首先从P-U表中通过商品编号查询到看到这个商品的用户，再从U-P表中查询到那些用户又看了那些商品的信息。在通过去重和统计算出这样子的功能。



#####实时类应用

和Facebook类似，我们也使用了hbase做为实时计算类项目的存储层。目前对内部己经上线了部分实时项目，比如**实时页面点击系统**，galaxy**实时交易推荐**以及**直播间**等内部项目，用户则是散布到公司内各部门的运营小二们。与facebook的puma不同的是淘宝使用了多种方式做实时计算层，比如galaxy是使用类似affa的actor模式处理交易数据，同时关联商品表等维度表计算排行(TopN)，而实时页面点击系统则是基于twitter开源的**storm**进行开发，后台通过TT获取实时的日志数据，计算流将中间结果以及动态维表持久化到hbase上，比如我们将rowkey设计为url+userid，并读出实时的数据，从而实现实时计算各个维度上的uv。




#####异步Hbase AsyncHBase

Multiple batches of 10k *new/updated* rows at any time to different tables
by different clients simultaneously. I want these multiple batches of
insertions to be done super fast. At the same time, I would like to be able
to scale up to 100k rows at a time (the goal). Now, I am building a cluster
of size 6 to 7 nodes.
If you're writing a multi-threaded client and you're going to have
many clients like this writing to HBase continuously, I recommend
writing your application with asynchbase
(http://github.com/stumbleupon/asynchbase) instead. It's an alternate
HBase client library I wrote and in my application it significantly
increased write throughput. It can easily push 150k updates per
second to a 20-node cluster – and then it's the local machine that's
CPU bound, not the HBase cluster (the local machine is a very slow VM
so it doesn't have a lot of horsepower). This client is especially
good for throughput oriented workloads and was written to be
thread-safe from the ground up (unlike HTable).