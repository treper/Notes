#####Hadoop config Notes

mapred.child.java.opts与mapreduce.map.memory.mb的区别

	Let's be more specific by a example. suppose that the mapred.child.java.opts is set to -Xmx800m and mapreduce.map.memory.mb is left as its default value 1024MB. When a map task run, the NodeManager will allocate a 1024MB container (decreasing the size of it's pool by that amount for the duration of task) and will launch the task JVM configured with a 800MB max heap size. Note that the JVM process will have a larger memory footprint that the heap size, and the overhead will depend on such thing as the native libs in use, the size of PermGen space, etc... The important thing is that the physical memory used by the JVM process, including any processes that it spawns, such as Streaming or Pipes processes, does not exceed it allocation(1024MB). If a container uses more memory than it has been allocated, then it may be terminated by the NodeManager and marked as failed.
 

yarn-site.xml设置

1. 手动计算规划

资源规划
操作系统留4G
finger-test1 40209-4096=36113
finger-test2 48289-4096=44193

2. 使用yarn-utils.py计算

finger-test1

		research@research:~/software$ python yarn-utils.py -c 24 -m 40 -d 5 -k False 
		 Using cores=24 memory=40GB disks=5 hbase=False
		 Profile: cores=24 memory=39936MB reserved=1GB usableMem=39GB disks=5
		 Num Container=9
		 Container Ram=4096MB
		 Used Ram=36GB
		 Unused Ram=1GB
		 yarn.scheduler.minimum-allocation-mb=4096
		 yarn.scheduler.maximum-allocation-mb=36864
		 yarn.nodemanager.resource.memory-mb=36864
		 mapreduce.map.memory.mb=4096
		 mapreduce.map.java.opts=-Xmx3276m
		 mapreduce.reduce.memory.mb=4096
		 mapreduce.reduce.java.opts=-Xmx3276m
		 yarn.app.mapreduce.am.resource.mb=4096
		 yarn.app.mapreduce.am.command-opts=-Xmx3276m
		 mapreduce.task.io.sort.mb=1638

如果设置disk为1

	research@research:~/software$ python yarn-utils.py -c 24 -m 40 -d 1 -k False  
	 Using cores=24 memory=40GB disks=1 hbase=False
	 Profile: cores=24 memory=39936MB reserved=1GB usableMem=39GB disks=1
	 Num Container=3
	 Container Ram=13312MB
	 Used Ram=39GB
	 Unused Ram=1GB
	 yarn.scheduler.minimum-allocation-mb=13312
	 yarn.scheduler.maximum-allocation-mb=39936
	 yarn.nodemanager.resource.memory-mb=39936
	 mapreduce.map.memory.mb=13312
	 mapreduce.map.java.opts=-Xmx10649m
	 mapreduce.reduce.memory.mb=13312
	 mapreduce.reduce.java.opts=-Xmx10649m
	 yarn.app.mapreduce.am.resource.mb=13312
	 yarn.app.mapreduce.am.command-opts=-Xmx10649m
	 mapreduce.task.io.sort.mb=5324

finger-test2

		research@research:~/software$ python yarn-utils.py -c 24 -m 48 -d 5 -k False  
		 Using cores=24 memory=48GB disks=5 hbase=False
		 Profile: cores=24 memory=43008MB reserved=6GB usableMem=42GB disks=5
		 Num Container=9
		 Container Ram=4608MB
		 Used Ram=40GB
		 Unused Ram=6GB
		 yarn.scheduler.minimum-allocation-mb=4608
		 yarn.scheduler.maximum-allocation-mb=41472
		 yarn.nodemanager.resource.memory-mb=41472
		 mapreduce.map.memory.mb=4608
		 mapreduce.map.java.opts=-Xmx3686m
		 mapreduce.reduce.memory.mb=4608
		 mapreduce.reduce.java.opts=-Xmx3686m
		 yarn.app.mapreduce.am.resource.mb=4608
		 yarn.app.mapreduce.am.command-opts=-Xmx3686m
		 mapreduce.task.io.sort.mb=1843

如果设置disk为1

	research@research:~/software$ python yarn-utils.py -c 24 -m 48 -d 1 -k False  
	 Using cores=24 memory=48GB disks=1 hbase=False
	 Profile: cores=24 memory=43008MB reserved=6GB usableMem=42GB disks=1
	 Num Container=3
	 Container Ram=14336MB
	 Used Ram=42GB
	 Unused Ram=6GB
	 yarn.scheduler.minimum-allocation-mb=14336
	 yarn.scheduler.maximum-allocation-mb=43008
	 yarn.nodemanager.resource.memory-mb=43008
	 mapreduce.map.memory.mb=14336
	 mapreduce.map.java.opts=-Xmx11468m
	 mapreduce.reduce.memory.mb=14336
	 mapreduce.reduce.java.opts=-Xmx11468m
	 yarn.app.mapreduce.am.resource.mb=14336
	 yarn.app.mapreduce.am.command-opts=-Xmx11468m
	 mapreduce.task.io.sort.mb=5734


相应的yarn-site.xml

finger-test1

	<?xml version="1.0"?>
	<configuration>
	
	  <property>
	    <name>yarn.resourcemanager.resource-tracker.address</name>
	    <value>finger-test1:8031</value>
	  </property>
	  <property>
	    <name>yarn.resourcemanager.address</name>
	    <value>finger-test1:8032</value>
	  </property>
	  <property>
	    <name>yarn.resourcemanager.scheduler.address</name>
	    <value>finger-test1:8030</value>
	  </property>
	  <property>
	    <name>yarn.resourcemanager.admin.address</name>
	    <value>finger-test1:8033</value>
	  </property>
	  <property>
	    <name>yarn.resourcemanager.webapp.address</name>
	    <value>finger-test1:8088</value>
	  </property>
	  <property>
	    <description>Classpath for typical applications.</description>
	    <name>yarn.application.classpath</name>
	    <value>$HADOOP_CONF_DIR,$HADOOP_COMMON_HOME/share/hadoop/common/*,
	  $HADOOP_COMMON_HOME/share/hadoop/common/lib/*,
	  $HADOOP_HDFS_HOME/share/hadoop/hdfs/*,$HADOOP_HDFS_HOME/share/hadoop/hdfs/lib/*,
	  $YARN_HOME/share/hadoop/yarn/*,$YARN_HOME/share/hadoop/yarn/lib/*,
	  $YARN_HOME/share/hadoop/mapreduce/*,$YARN_HOME/share/hadoop/mapreduce/lib/*</value>
	  </property>
	  <property>
	    <name>yarn.nodemanager.aux-services</name>
	    <value>mapreduce.shuffle</value>
	  </property>
	  <property>
	    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
	    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
	  </property>
	
	  <property>
	    <name>yarn.nodemanager.local-dirs</name>
	    <value>/home/mps/cdh/local</value>
	  </property>
	  <property>
	    <name>yarn.nodemanager.log-dirs</name>
	    <value>/home/mps/cdh/logs</value>
	  </property>
	  <property>
	    <description>Where to aggregate logs</description>
	    <name>yarn.nodemanager.remote-app-log-dir</name>
	    <value>/home/mps/cdh/logs</value>
	  </property>
	
	  <property>
	    <name>yarn.app.mapreduce.am.staging-dir</name>
	    <value>/home/mps/cdh/users</value>
	  </property>
	
	<property>
	  <name>yarn.nodemanager.resource.memory-mb</name>
	  <value>39936</value>
	</property>
	<property>
	  <name>yarn.scheduler.minimum-allocation-mb</name>
	  <value>13312</value>
	</property>
	<property>
	  <name>yarn.scheduler.maximum-allocation-mb</name>
	  <value>39936</value>
	</property>
	<property>
	  <name>yarn.app.mapreduce.am.resource.mb</name>
	  <value>13312</value>
	</property>
	<property>
	  <name>yarn.app.mapreduce.am.command-opts</name>
	  <value>-Xmx10649m</value>
	</property>
	</configuration>

finger-test2

	<?xml version="1.0"?>
	<configuration>
	
	  <property>
	    <name>yarn.resourcemanager.resource-tracker.address</name>
	    <value>finger-test1:8031</value>
	  </property>
	  <property>
	    <name>yarn.resourcemanager.address</name>
	    <value>finger-test1:8032</value>
	  </property>
	  <property>
	    <name>yarn.resourcemanager.scheduler.address</name>
	    <value>finger-test1:8030</value>
	  </property>
	  <property>
	    <name>yarn.resourcemanager.admin.address</name>
	    <value>finger-test1:8033</value>
	  </property>
	  <property>
	    <name>yarn.resourcemanager.webapp.address</name>
	    <value>finger-test1:8088</value>
	  </property>
	  <property>
	    <description>Classpath for typical applications.</description>
	    <name>yarn.application.classpath</name>
	    <value>$HADOOP_CONF_DIR,$HADOOP_COMMON_HOME/share/hadoop/common/*,
	  $HADOOP_COMMON_HOME/share/hadoop/common/lib/*,
	  $HADOOP_HDFS_HOME/share/hadoop/hdfs/*,$HADOOP_HDFS_HOME/share/hadoop/hdfs/lib/*,
	  $YARN_HOME/share/hadoop/yarn/*,$YARN_HOME/share/hadoop/yarn/lib/*,
	  $YARN_HOME/share/hadoop/mapreduce/*,$YARN_HOME/share/hadoop/mapreduce/lib/*</value>
	  </property>
	  <property>
	    <name>yarn.nodemanager.aux-services</name>
	    <value>mapreduce.shuffle</value>
	  </property>
	  <property>
	    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
	    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
	  </property>
	
	  <property>
	    <name>yarn.nodemanager.local-dirs</name>
	    <value>/home/mps/cdh/local</value>
	  </property>
	  <property>
	    <name>yarn.nodemanager.log-dirs</name>
	    <value>/home/mps/cdh/logs</value>
	  </property>
	  <property>
	    <description>Where to aggregate logs</description>
	    <name>yarn.nodemanager.remote-app-log-dir</name>
	    <value>/home/mps/cdh/logs</value>
	  </property>
	
	  <property>
	    <name>yarn.app.mapreduce.am.staging-dir</name>
	    <value>/home/mps/cdh/users</value>
	  </property>
	<property>
	  <name>yarn.nodemanager.resource.memory-mb</name>
	  <value>43008</value>
	</property>
	<property>
	  <name>yarn.scheduler.minimum-allocation-mb</name>
	  <value>14336</value>
	</property>
	<property>
	  <name>yarn.scheduler.maximum-allocation-mb</name>
	  <value>43008</value>
	</property>
	<property>
	  <name>yarn.app.mapreduce.am.resource.mb</name>
	  <value>14336</value>
	</property>
	<property>
	  <name>yarn.app.mapreduce.am.command-opts</name>
	  <value>-Xmx11468m</value>
	</property>
	</configuration>

相应的mapred-site.xml

finger-test1

	<?xml version="1.0"?>
	<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
	<configuration>
	    <property>
	        <name>mapreduce.shuffle.port</name>
	        <value>8017</value>
	    </property>
	
	    <property>
	        <name>mapreduce.framework.name</name>
	        <value>yarn</value>
	    </property>
	
	    <property>
	        <name>mapreduce.jobhistory.address</name>
	        <value>finger-test1:10020</value>
	    </property>
	
	    <property>
	        <name>mapreduce.jobhistory.webapp.address</name>
	        <value>finger-test1:19888</value>
	    </property>
	
	<property>
	  <name>mapreduce.map.memory.mb</name>
	  <value>13312</value>
	</property>
	<property>
	  <name>mapreduce.map.java.opts</name>
	  <value>-Xmx10649m</value>
	</property>
	<property>
	  <name>mapreduce.reduce.memory.mb</name>
	  <value>13312</value>
	</property>
	<property>
	  <name>mapreduce.reduce.java.opts</name>
	  <value>-Xmx10649m</value>
	</property>
	<property>
	  <name>mapreduce.task.io.sort.mb</name>
	  <value>1000</value>
	</property>
	</configuration>

finger-test2

	<?xml version="1.0"?>
	<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
	<configuration>
	    <property>
	        <name>mapreduce.shuffle.port</name>
	        <value>8017</value>
	    </property>
	
	    <property>
	        <name>mapreduce.framework.name</name>
	        <value>yarn</value>
	    </property>
	
	    <property>
	        <name>mapreduce.jobhistory.address</name>
	        <value>finger-test1:10020</value>
	    </property>
	
	    <property>
	        <name>mapreduce.jobhistory.webapp.address</name>
	        <value>finger-test1:19888</value>
	    </property>
	<property>
	  <name>mapreduce.map.memory.mb</name>
	  <value>14336</value>
	</property>
	<property>
	  <name>mapreduce.map.java.opts</name>
	  <value>-Xmx11468m</value>
	</property>
	<property>
	  <name>mapreduce.reduce.memory.mb</name>
	  <value>14336</value>
	</property>
	<property>
	  <name>mapreduce.reduce.java.opts</name>
	  <value>-Xmx11468m</value>
	</property>
	<property>
	  <name>mapreduce.task.io.sort.mb</name>
	  <value>1000</value>
	</property>
	</configuration>

hadoop-1.2的配置文件备份

`core-site.xml`

	<?xml version="1.0"?>
	<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
	
	<!-- Put site-specific property overrides in this file. -->
	<configuration>
	<property>
	  <name>hadoop.mydata.dir</name>
	  <value>/home/mps/hdfs</value>
	  <description>A base for other directories.${user.name} </description>
	</property>
	
	<property>
	  <name>hadoop.tmp.dir</name>
	  <value>/home/mps/hdfs/tmp/hadoop-${user.name}</value>
	  <description>A base for other temporary directories.</description>
	</property>
	
	<property>
	  <name>fs.default.name</name>
	  <value>hdfs://finger-test2:54310</value>
	  <description></description>
	</property>
	
	</configuration>

`hdfs-site.xml`


	<?xml version="1.0"?>
	<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
	
	<!-- Put site-specific property overrides in this file. -->
	<configuration>
	<property>
	  <name>hadoop.mydata.dir</name>
	  <value>/home/mps/hdfs</value>
	  <description>A base for other directories.${user.name} </description>
	</property>
	
	<property>
	  <name>hadoop.tmp.dir</name>
	  <value>/home/mps/hdfs/tmp/hadoop-${user.name}</value>
	  <description>A base for other temporary directories.</description>
	</property>
	
	<property>
	  <name>fs.default.name</name>
	  <value>hdfs://finger-test2:54310</value>
	  <description>The name of the default file system.  A URI whose
	  scheme and authority determine the FileSystem implementation.  The
	  uri's scheme determines the config property (fs.SCHEME.impl) naming
	  the FileSystem implementation class.  The uri's authority is used to
	  determine the host, port, etc. for a filesystem.</description>
	</property>
	
	</configuration>

`mapred-site.xml`

	<?xml version="1.0"?>
	<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
	
	<!-- Put site-specific property overrides in this file. -->
	<configuration>
	<property>
	  <name>mapred.job.tracker</name>
	  <value>finger-test2:54311</value>
	  <description>The host and port that the MapReduce job tracker runs
	  at.  If "local", then jobs are run in-process as a single map
	  and reduce task.
	  </description>
	</property>
	
	<property>
	  <name>mapred.local.dir</name>
	  <value>${hadoop.tmp.dir}/mapred/local</value>
	  <description>The local directory where MapReduce stores intermediate
	  data files.  May be a comma-separated list of
	  directories on different devices in order to spread disk i/o.
	  Directories that do not exist are ignored.
	  </description>
	</property>
	
	<property>
	  <name>mapred.system.dir</name>
	  <value>${hadoop.mydata.dir}/mapred/system</value>
	  <description>The directory where MapReduce stores control files.
	  </description>
	</property>
	<property>
	  <name>mapred.tasktracker.map.tasks.maximum</name>
	  <value>18</value>
	  <description>The maximum number of map tasks that will be run
	  simultaneously by a task tracker.vary it depending on your hardware
	  </description>
	</property>
	
	<property>
	  <name>mapred.tasktracker.reduce.tasks.maximum</name>
	  <value>12</value>
	  <description>The maximum number of reduce tasks that will be run
	  simultaneously by a task tracker.vary it depending on your hardware
	  </description>
	</property>
	<property>
	  <name>mapred.task.timeout</name>
	  <value>1800000</value> <!-- 30 minutes -->
	</property>
	<property>
	    <name>mapreduce.jobtracker.heartbeat.interval.min</name>
	    <value>300</value>
	</property>
	<property>
	    <name>tasktracker.http.threads</name>
	    <value>24</value>
	</property>
	<property>
	    <name>mapred.compress.map.output</name>
	    <value>true</value>
	</property>
	<property>
	    <name>mapred.job.reuse.jvm.num.tasks</name>
	    <value>-1</value>
	</property>
	<property>
	<name>mapred.user.jobconf.limit</name>
	<value>209715200</value>
	</property>
	<property>
	    <name>mapred.child.java.opts</name>
	    <value>-Xmx10000m</value>
	</property>
	<property>
	    <name>mapred.map.child.java.opts</name>
	    <value>-Xmx10000m</value>
	</property>
	<property>
	    <name>mapred.reduce.child.java.opts</name>
	    <value>-Xmx10000m</value>
	</property>
	</configuration>


#####Reference


[Determine YARN and MapReduce Memory Configuration Settings](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.9.1/bk_installing_manually_book/content/rpm-chap1-11.html)

[Hadoop YARN配置参数剖析(1)—RM与NM相关参数](http://dongxicheng.org/mapreduce-nextgen/hadoop-yarn-configurations-resourcemanager-nodemanager/)

[yarn内存配置指南](http://blog.csdn.net/yangfei001/article/details/37766747)

#####Performance Tunning Notes

[http://www.idryman.org/blog/2014/03/05/hadoop-performance-tuning-best-practices/](http://www.idryman.org/blog/2014/03/05/hadoop-performance-tuning-best-practices/)

降低disk spill

	<property>
	    <name>mapred.compress.map.output</name>
	    <value>true</value>
	</property>
	<property>
	    <name>mapred.map.output.compression.codec</name>
	    <value>com.hadoop.compression.lzo.LzoCodec</value>
	</property>
	<property>
	    <name>io.sort.mb</name>
	    <value>800</value>
	</property>

Mapper调优

jvm重用

	mapred.job.reuse.jvm.num.tasks=-1

关掉推测执行避免冗余map,task

	mapred.map.tasks.speculative.execution

由于磁盘数目只有一个,多个mapreducer可能竞争读磁盘,导致速度变慢，现象free -m还有很多可用内存但是速度还是好慢.

**Hadoop 1 Sample Config File**
	
	<property>
	    <name>mapred.compress.map.output</name>
	    <value>true</value>
	</property>
	<property>
	    <name>mapred.map.output.compression.codec</name>
	    <value>com.hadoop.compression.lzo.LzoCodec</value>
	</property>
	<property>
	    <name>io.sort.mb</name>
	    <value>800</value>
	</property>
	<property>
	    <name>mapred.job.reuse.jvm.num.tasks</name>
	    <value>-1</value>
	</property>
	<property>
	    <name>mapred.max.split.size</name>
	    <value>268435456</value>
	</property>
	<property>
	    <name>mapred.min.split.size</name>
	    <value>134217728</value>
	</property>

其他codecs

	org.apache.hadoop.io.compress.SnappyCodec

**Hadoop 2 Sample Config File**

	<property>
	  <name>mapreduce.map.output.compress</name>  
	  <value>true</value>
	</property>
	<property>
	  <name>mapred.map.output.compress.codec</name>  
	  <value>org.apache.hadoop.io.compress.SnappyCodec</value>
	</property>


	For MRv1:
	<property>
	  <name>mapred.compress.map.output</name>  
	  <value>true</value>
	</property>
	<property>
	  <name>mapred.map.output.compression.codec</name>  
	  <value>org.apache.hadoop.io.compress.SnappyCodec</value>
	</property>
	For YARN:
	<property>
	  <name>mapreduce.map.output.compress</name>  
	  <value>true</value>
	</property>
	<property>
	  <name>mapred.map.output.compress.codec</name>  
	  <value>org.apache.hadoop.io.compress.SnappyCodec</value>
	</property>

其中关于LZO压缩的设置参考

[http://hsiamin.com/posts/2014/05/03/enable-lzo-compression-on-hadoop-pig-and-spark/](http://hsiamin.com/posts/2014/05/03/enable-lzo-compression-on-hadoop-pig-and-spark/)


Hadoop 1和Hadoop 2新旧参数对照表

[http://hadoop.apache.org/docs/r2.3.0/hadoop-project-dist/hadoop-common/DeprecatedProperties.html](http://hadoop.apache.org/docs/r2.3.0/hadoop-project-dist/hadoop-common/DeprecatedProperties.html)

Hadoop 主要参数的设置参考

[http://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/ClusterSetup.html](http://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/ClusterSetup.html)

