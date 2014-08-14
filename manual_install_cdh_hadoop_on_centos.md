#####Centos 5.7 手动安装CDH hadoop 4.7

**CDH相关资源下载链接**

[http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH4/latest/CDH-Version-and-Packaging-Information/cdhvd_topic_3.html](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH4/latest/CDH-Version-and-Packaging-Information/cdhvd_topic_3.html)

[http://archive.cloudera.com/cdh4/cdh/4/](http://archive.cloudera.com/cdh4/cdh/4/)


1、硬件资源规划

|ip|host|role|
|:-:|:-:|:-:|
|10.25.13.152|finger-test2|NameNode,ResourceManager,SecondaryNameNode,DataNode,NodeManager|
|10.25.13.151|finger-test1|DataNode,NodeManager|

**免密码ssh**

	ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
	cat ~/.ssh/id_rsa.pub >> /.ssh/authorized_keys

	ssh localhost


**部署步骤**

tar -xzf hadoop-2.0.0-cdh4.7.0.tar.gz 

1. core-site.xml

		<?xml version="1.0" encoding="UTF-8"?>
		<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
		 
		<configuration>
		    <!--fs.default.name for MRV1 ,fs.defaultFS for MRV2(yarn) -->
		    <property>
		        <name>fs.default.name</name>
		        <!--这个地方的值要和hdfs-site.xml文件中的dfs.federation.nameservices一致-->
		        <value>hdfs://finger-test2:9100</value>
		    </property>
		    <property>
		        <name>fs.trash.interval</name>
		        <value>0</value>
		    </property>
		    <property>
		        <name>fs.trash.checkpoint.interval</name>
		        <value>10080</value>
		    </property>
		 
		</configuration>

2. hdfs-site.xml

		<?xml version="1.0" encoding="UTF-8"?>
		<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
		<configuration>
		    <property>
		        <name>dfs.replication</name>
		        <value>1</value>
		    </property>
		 
		    <property>
		        <name>hadoop.tmp.dir</name>
		        <value>/home/mps/cdh/tmp</value>
		    </property>
		 
		    <property>
		        <name>dfs.namenode.http-address</name>
		        <value>finger-test2:50070</value>
		    </property>
		 
		    <property>
		        <name>dfs.namenode.secondary.http-address</name>
		        <value>finger-test2:50090</value>
		    </property>
		 
		    <property>
		        <name>dfs.webhdfs.enabled</name>
		        <value>true</value>
		    </property>
		</configuration>

3. masters

		finger-test2

4. slaves

		finger-test1
		finger-test2

5. mapred-site.xml

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
		        <value>finger-test2:10020</value>
		    </property>
		 
		    <property>
		        <name>mapreduce.jobhistory.webapp.address</name>
		        <value>finger-test2:19888</value>
		    </property>
		</configuration>

6. yarn-site.xml

		<?xml version="1.0"?>
		<configuration>
		
		  <property>
		    <name>yarn.resourcemanager.resource-tracker.address</name>
		    <value>finger-test2:8031</value>
		  </property>
		  <property>
		    <name>yarn.resourcemanager.address</name>
		    <value>finger-test2:8032</value>
		  </property>
		  <property>
		    <name>yarn.resourcemanager.scheduler.address</name>
		    <value>finger-test2:8030</value>
		  </property>
		  <property>
		    <name>yarn.resourcemanager.admin.address</name>
		    <value>finger-test2:8033</value>
		  </property>
		  <property>
		    <name>yarn.resourcemanager.webapp.address</name>
		    <value>finger-test2:8088</value>
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
		
		</configuration>


7. .bash_profile

		export LANG=zh_CN.utf8
		 
		export JAVA_HOME=/usr/local/jdk1.7.0_01
		export JRE_HOME=$JAVA_HOME/jre
		export CLASSPATH=$CLASSPATH:.:$JAVA_HOME/lib:$JRE_HOME/lib
		 
		export HADOOP_HOME=/home/mps/software/hadoop-2.0.0-cdh4.7.0
		 
		export HADOOP_MAPRED_HOME=${HADOOP_HOME}
		export HADOOP_COMMON_HOME=${HADOOP_HOME}
		export HADOOP_HDFS_HOME=${HADOOP_HOME}
		export YARN_HOME=${HADOOP_HOME}
		export HADOOP_YARN_HOME=${HADOOP_HOME}
		export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
		export HDFS_CONF_DIR=${HADOOP_HOME}/etc/hadoop
		export YARN_CONF_DIR=${HADOOP_HOME}/etc/hadoop
		 
		export PATH=$CLASSPATH:$HADOOP_HOME:.:$PATH:$HOME/bin:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HBASE_HOME/bin:$HIVE_HOME/bin
		 
		alias hd="cd /home/mps/software/hadoop-2.0.0-cdh4.7.0"

8. 运行测试

格式化

	hadoop namenode -format

finger-test2上启动hdfs：
	
	start-dfs.sh

finger-test2启动mapreduce：

	start-yarn.sh

finger-test2上启动historyserver：

	mr-jobhistory-daemon.sh start historyserver

关闭

	stop-all.sh
	mr-jobhistory-daemon.sh stop historyserver

由于finger-test1硬盘较充足,中途改用finger-test1为master,命令行运行结果，注意看各个node启动情况

	[mps@finger-test1 hadoop-2.0.0-cdh4.7.0]$ start-dfs.sh                   
	14/07/16 15:49:49 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
	Starting namenodes on [finger-test1]
	finger-test1: starting namenode, logging to /home/mps/software/hadoop-2.0.0-cdh4.7.0/logs/hadoop-mps-namenode-finger-test1.out
	finger-test1: starting datanode, logging to /home/mps/software/hadoop-2.0.0-cdh4.7.0/logs/hadoop-mps-datanode-finger-test1.out
	finger-test2: starting datanode, logging to /home/mps/software/hadoop-2.0.0-cdh4.7.0/logs/hadoop-mps-datanode-finger-test2.out
	Starting secondary namenodes [finger-test1]
	finger-test1: starting secondarynamenode, logging to /home/mps/software/hadoop-2.0.0-cdh4.7.0/logs/hadoop-mps-secondarynamenode-finger-test1.out
	14/07/16 15:50:04 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
	[mps@finger-test1 hadoop-2.0.0-cdh4.7.0]$ jps
	20802 NameNode
	21344 Jps
	20966 DataNode
	21186 SecondaryNameNode
	[mps@finger-test1 hadoop-2.0.0-cdh4.7.0]$ start-yarn.sh
	starting yarn daemons
	starting resourcemanager, logging to /home/mps/software/hadoop-2.0.0-cdh4.7.0/logs/yarn-mps-resourcemanager-finger-test1.out
	finger-test2: starting nodemanager, logging to /home/mps/software/hadoop-2.0.0-cdh4.7.0/logs/yarn-mps-nodemanager-finger-test2.out
	finger-test1: starting nodemanager, logging to /home/mps/software/hadoop-2.0.0-cdh4.7.0/logs/yarn-mps-nodemanager-finger-test1.out
	[mps@finger-test1 hadoop-2.0.0-cdh4.7.0]$ jps
	21851 Jps
	20802 NameNode
	21418 ResourceManager
	21562 NodeManager
	20966 DataNode
	21186 SecondaryNameNode


任务运行日志


任务的日志在

	/home/mps/cdh/logs/application_1405497023620_0025/container_1405497023620_0025_01_000047/各个slot的日志
	
其他

	hadoop fs -ls -R　/

	-rw-rw----   1 mps supergroup      12418 2014-07-16 17:00 /home/mps/cdh/users/history/done_intermediate/mps/job_1405497023620_0007-1405501212653-mps-simjoin%2D1.0.jar-1405501235785-0-0-FAILED-default.jhist
	-rw-rw----   1 mps supergroup        337 2014-07-16 17:00 /home/mps/cdh/users/history/done_intermediate/mps/job_1405497023620_0007.summary
	-rw-rw----   1 mps supergroup      72924 2014-07-16 17:00 /home/mps/cdh/users/history/done_intermediate/mps/job_1405497023620_0007_conf.xml

**finger-test2中.bash_profile的备份**

	# .bash_profile
	
	# Get the aliases and functions
	if [ -f ~/.bashrc ]; then
	        . ~/.bashrc
	fi
	
	# User specific environment and startup programs
	#export CC=/home/mps/software/gcc-4.8.0/bin/gcc
	#export CXX=/home/mps/software/gcc-4.8.0/bin/g++
	export JAVA_HOME=/usr/local/jdk1.7.0_01/
	export LD_LIBRARY_PATH=/home/mps/software/gcc-4.8.0/lib:/home/mps/software/gcc-4.8.0/lib64/:/usr/local/lib:/usr/lib:/home/is_admin/codevideo-12.0-linux-exe/codevideo/lib:/home/is_admin/mysql/lib:/home/is_admin/gsl/lib:/home/is_admin/lshkit/lib:/home/is_admin/libunwind-1.0.1/lib:/home/is_admin/graphviz/lib:/home/is_admin/ghostscript/lib:/home/is_admin/libssh2/lib:/home/is_admin/jre-1.7.0/lib:/home/is_admin/fftw-3.3.1/lib:/home/is_admin/software/lib:/home/is_admin/protobuf/lib
	export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/is_admin/software/ffmpeg/libavcodec/
	export GSL_DIR=/home/is_admin/gsl
	export CPUPROFILE=/home/is_admin/MRPlatform/cpuprofile.out
	export SCALA_HOME=/home/mps/software/scala-2.9.3
	export GIRAPH_HOME=/home/mps/software/giraph
	#HADOOP_HOME=/home/mps/software/hadoop-1.1.2
	export SQOOP_HOME=/home/mps/software/sqoop-1.4.4.bin__hadoop-1.0.0
	HADOOP_HOME=/home/mps/software/hadoop-1.2.0
	export HADOOP_HOME
	export HADOOP_HOME_WARN_SUPPRESS=1
	
	#export PATH=$PATH:/etc/haproxy/sbin/:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
	#export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$HADOOP_LIB/native/libhadoop.so
	PATH=/usr/sbin:$PATH:$HOME/bin:/home/mps/.rvm/bin:/home/mps/software/apache-ant-1.9.1/bin:$JAVA_HOME/bin:$MONGODB_HOME/bin:$ZOOKEEPER_INSTALL/bin:$HADOOP_HOME/bin:$SCALA_HOME/bin:$HIVE_HOME/bin:$MAHOUT_HOME/bin:$M2_HOME/bin:/home/is_admin/bin:/home/is_admin/cmake/bin:/home/is_admin/mysql/bin:/home/is_admin/vim/bin:/home/is_admin/emacs/bin:/home/is_admin/ctags/bin:/home/is_admin/gsl/bin:/home/is_admin/libunwind-1.0.3/bin:/home/is_admin/perftools/bin:/home/is_admin/graphviz/bin:/home/is_admin/ghostscript/bin:/home/is_admin/jre-1.7.0/bin:/home/is_admin/fftw-3.3.1/bin:$SQOOP_HOME/bin
	export PATH
	export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
	export http_proxy="http://localhost:80"
	export https_proxy="http://localhost:80"
	#export PYTHONPATH=/home/mps/python
	#stty cs8 -istrip
	#export LANG=zh_CN.UTF-8
	#export CC=gcc44
	#export CXX=g++44
	export TESSDATA_PREFIX=/home/mps/software/tesseract-ocr

**新的bash_profile**


	export JAVA_HOME=/usr/local/jdk1.7.0_01/
	export LD_LIBRARY_PATH=/home/mps/software/gcc-4.8.0/lib:/home/mps/software/gcc-4.8.0/lib64/:/usr/local/lib:/usr/lib:/home/is_admin/codevideo-12.0-linux-exe/codevideo/lib:/home/is_admin/mysql/lib:/home/is_admin/gsl/lib:/home/is_admin/lshkit/lib:/home/is_admin/libunwind-1.0.1/lib:/home/is_admin/graphviz/lib:/home/is_admin/ghostscript/lib:/home/is_admin/libssh2/lib:/home/is_admin/jre-1.7.0/lib:/home/is_admin/fftw-3.3.1/lib:/home/is_admin/software/lib:/home/is_admin/protobuf/lib
	export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/is_admin/software/ffmpeg/libavcodec/
	export GSL_DIR=/home/is_admin/gsl
	export CPUPROFILE=/home/is_admin/MRPlatform/cpuprofile.out
	export SCALA_HOME=/home/mps/software/scala-2.9.3
	export GIRAPH_HOME=/home/mps/software/giraph
	#HADOOP_HOME=/home/mps/software/hadoop-1.1.2
	export SQOOP_HOME=/home/mps/software/sqoop-1.4.4.bin__hadoop-1.0.0
	#HADOOP_HOME=/home/mps/software/hadoop-1.2.0
	#export HADOOP_HOME
	#export HADOOP_HOME_WARN_SUPPRESS=1
	export LANG=zh_CN.utf8
	
	export JAVA_HOME=/usr/local/jdk1.7.0_01
	export JRE_HOME=$JAVA_HOME/jre
	export CLASSPATH=$CLASSPATH:.:$JAVA_HOME/lib:$JRE_HOME/lib
	
	export HADOOP_HOME=/home/mps/software/hadoop-2.0.0-cdh4.7.0
	
	export HADOOP_MAPRED_HOME=${HADOOP_HOME}
	export HADOOP_COMMON_HOME=${HADOOP_HOME}
	export HADOOP_HDFS_HOME=${HADOOP_HOME}
	export YARN_HOME=${HADOOP_HOME}
	export HADOOP_YARN_HOME=${HADOOP_HOME}
	export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
	export HDFS_CONF_DIR=${HADOOP_HOME}/etc/hadoop
	export YARN_CONF_DIR=${HADOOP_HOME}/etc/hadoop
	
	export PATH=$CLASSPATH:$HADOOP_HOME:.:$PATH:$HOME/bin:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HBASE_HOME/bin:$HIVE_HOME/bin
	
	alias hd="cd /home/mps/software/hadoop-2.0.0-cdh4.7.0"
	#export PATH=$PATH:/etc/haproxy/sbin/:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
	#export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$HADOOP_LIB/native/libhadoop.so
	PATH=/usr/sbin:$PATH:$HOME/bin:/home/mps/.rvm/bin:/home/mps/software/apache-ant-1.9.1/bin:$JAVA_HOME/bin:$MONGODB_HOME/bin:$ZOOKEEPER_INSTALL/bin:$HADOOP_HOME/bin:$SCALA_HOME/bin:$HIVE_HOME/bin:$MAHOUT_HOME/bin:$M2_HOME/bin:/home/is_admin/bin:/home/is_admin/cmake/bin:/home/is_admin/mysql/bin:/home/is_admin/vim/bin:/home/is_admin/emacs/bin:/home/is_admin/ctags/bin:/home/is_admin/gsl/bin:/home/is_admin/libunwind-1.0.3/bin:/home/is_admin/perftools/bin:/home/is_admin/graphviz/bin:/home/is_admin/ghostscript/bin:/home/is_admin/jre-1.7.0/bin:/home/is_admin/fftw-3.3.1/bin:$SQOOP_HOME/bin
	export PATH
	export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
	export http_proxy="http://localhost:80"
	export https_proxy="http://localhost:80"
	#export PYTHONPATH=/home/mps/python
	#stty cs8 -istrip
	#export LANG=zh_CN.UTF-8
	#export CC=gcc44
	#export CXX=g++44
	export TESSDATA_PREFIX=/home/mps/software/tesseract-ocr
