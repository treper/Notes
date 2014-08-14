#####通用虚拟机box制作
1. vagrant启动base之后安装所需要的软件，
 
	配置环境变量

		export JAVA_HOME=/home/vagrant/software/jdk1.7.0_60
		export MAVEN_HOME=/home/vagrant/software/apache-maven-3.2.2
		export TOMCAT_HOME=/home/vagrant/software/apache-tomcat-7.0.26
		export KAFKA_HOME=/home/vagrant/software/kafka_2.8.0-0.8.1.1
		export ZOOKEEPER_HOME=/home/vagrant/software/zookeeper-3.4.6
		export STORM_HOME=/home/vagrant/software/apache-storm-0.9.2-incubating
		export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin:$TOCAT_HOME/bin:$KAFKA_HOME/bin:$ZOOKEEPER_HOME/bin:$STORM_HOME/bin
	
	打包新的box
		
		vagrant package --base master --output bigdata.box

2. 启动
#####vagrant box虚拟机的单机配置

1. samba设置

		sudo vim /etc/samba/smb.conf
	以下为修改的地方

		[global]
		hosts allow = 10.5.20.75

		[public]
		comment = Public Stuff
		path = /home/vagrant
		public = yes
		writable = yes
		valid users = vagrant
	然后：

		smbpasswd -a vagrant #添加秘密 会提示输入密码
		smbpasswd vagrant#可以改密码
		sudo /etc/init.d/iptables stop
		sudo service smb start

2. 桥接网络文件

		Vagrant.configure("2") do |config|
		  config.vm.define :master do |master|
		    master.vm.provider "virtualbox" do |v|
		          v.customize ["modifyvm", :id, "--name", "master", "--memory", "2048", "--cpus","2"]
		    end
		    master.vm.box = "base"
		    master.vm.hostname = "master"
		    master.vm.network :public_network
		  end
		end

3. 加载同一镜像但启动多个虚拟机的Vagrantfile:

		Vagrant.configure("2") do |config|
		  config.vm.define :master do |master|
		    master.vm.provider "virtualbox" do |v|
		          v.customize ["modifyvm", :id, "--name", "master", "--memory", "2048"]
		    end
		    master.vm.box = "base"
		    master.vm.hostname = "master"
		    master.vm.network :private_network, ip: "33.33.33.09"
		  end
		
		  config.vm.define :zookeeper1 do |zookeeper1|
		    zookeeper1.vm.provider "virtualbox" do |v|
		          v.customize ["modifyvm", :id, "--name", "zookeeper1", "--memory", "2048"]
		    end
		    zookeeper1.vm.box = "base"
		    zookeeper1.vm.hostname = "zookeeper1"
		    zookeeper1.vm.network :private_network, ip: "33.33.33.10"
		  end
		
		  config.vm.define :slave1 do |slave1|
		    slave1 .vm.provider "virtualbox" do |v|
		          v.customize ["modifyvm", :id, "--name", "slave1", "--memory", "2048"]
		    end
		    slave1 .vm.box = "base"
		    slave1 .vm.hostname = "slave1"
		    slave1 .vm.network :private_network, ip: "33.33.33.11"
		  end
		
		  config.vm.define :slave2 do |slave2|
		    slave2 .vm.provider "virtualbox" do |v|
		          v.customize ["modifyvm", :id, "--name", "slave2", "--memory", "2048"]
		    end
		    slave2 .vm.box = "base"
		    slave2 .vm.hostname = "slave2"
		    slave2 .vm.network :private_network, ip: "33.33.33.12"
		  end
		end
#####zookeeper集群配置

1. 单机部署

2. 分布式部署 3台虚拟机

		tickTime=2000
		initLimit=5
		syncLimit=5
		dataDir=/home/vagrant/data/zookeeper/data  # 目录需要手工建立，存放 zk 数据，主要是快照
		clientPort=2181
		# dataLogDir事务日志存放目录，最好配置，事务日志的写入速度严重影响zookeeper的性能
		dataLogDir=/home/vagrant/data/zookeeper/datalog
		server.1=33.33.33.101:2888:3888
		server.2=33.33.33.102:2888:3888
		server.3=33.33.33.103:2888:3888


	Vagrantfile:

		Vagrant.configure("2") do |config|
		  config.vm.define :zk1 do |zk1|
		    zk1.vm.provider "virtualbox" do |v|
		          v.customize ["modifyvm", :id, "--name", "zk1", "--memory", "512"]
		    end
		    zk1.vm.box = "bigdata"
		    zk1.vm.hostname = "zk1"
		    zk1.vm.network :private_network, ip: "33.33.33.101"
		  end
		
		  config.vm.define :zk2 do |zk2|
		    zk2.vm.provider "virtualbox" do |v|
		          v.customize ["modifyvm", :id, "--name", "zk2", "--memory", "256"]
		    end
		    zk2.vm.box = "bigdata"
		    zk2.vm.hostname = "zk2"
		    zk2.vm.network :private_network, ip: "33.33.33.102"
		  end
		
		  config.vm.define :zk3 do |zk3|
		    zk3.vm.provider "virtualbox" do |v|
		          v.customize ["modifyvm", :id, "--name", "zk3", "--memory", "256"]
		    end
		    zk3.vm.box = "bigdata"
		    zk3.vm.hostname = "zk3"
		    zk3.vm.network :private_network, ip: "33.33.33.103"
		  end
		end

	启动设置

	设置开机关闭防火墙和启动samba服务：在`/etc/rc.d/rc.local`添加：

		/etc/init.d/iptables stop
		service smb start 


2.1 启动虚拟机可能遇到的问题

2.1.1 问题1

		Stderr: VBoxManage: error: Runtime error opening '/home/mps/VirtualBox VMs/bigdata_zk1_1404381808488_4464/bigdata_zk1_1404381808488_4464.vbox-tmp' for reading: -102(File not found.)

	暴力解决方法：清理所有

		rm -rf /home/mps/VirtualBox VMs/*
		vagrant destroy


2.1.2 问题2
		
		zk1: SSH auth method: private key
		zk1: Warning: Connection timeout. Retrying...

	解决：

		在.vagrant.d目录下找到了文件insecure_private_key，在ssh验证时就是使用的这个私钥验证的，将该文件内容替换为我的私钥，然后将我的公钥加入到vagrant主机里的/home/vagrant/.ssh/authorized_keys里面，关闭vagrant实例， 重新打开，已经不会出现验证错误了(可能会出现1到2次的timeout，但是会自动跳过的)。

2.1.3 问题3
	
	用桥接可能无法配置ip，解释

		Bridged network simply don't work on some routers. Vagrant can't really detect this so you'll have to check your router. The original issue should be fixed in git.

	暂时的解决是使用 `private network`

2.1.4 问题4

		The following SSH command responded with a non-zero exit status.
		Vagrant assumes that this means the command failed!
		
		/sbin/ip addr flush dev eth1 2> /dev/null
		
		Stdout from the command:
		
		
		
		Stderr from the command:
		
		stdin: is not a tty

解决：删除`/etc/udev/rules.d/70-persistent-net.rules`，注意这个文件在每次重新从box加载后又会出现，每次都要删除一次。

		sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
		vagrant reload zk1

2.1.5 问题5

	打包的box损坏了？似乎一个一个启动更容易成功，或从每个虚拟机启动另外一个文件夹
	
2.1.6 问题6

    zktest1: Warning: Authentication failure. Retrying...
    zktest1: Warning: Authentication failure. Retrying...
    zktest1: Warning: Authentication failure. Retrying...
    zktest1: Warning: Authentication failure. Retrying...

	将宿主机的id_rsa.pub加入到~/.ssh/authorized_keys里

#####zookeeper集群的启动

#####zookeeper集群的测试
#####storm集群配置与测试

1.注意事项：

在每个配置项前面必须留有空格，否则会无法识别。

2.启动
		
		zkServer.sh start
		zkServer.sh status
		
		storm nimbus >/dev/null 2>&1 &
		storm supervisor >/dev/null 2>&1 &
		storm ui >/dev/null 2>&1 &
		
		ssh -L 8001:10.25.13.152:8080 -g tangning@10.25.251.101 -N
		http://10.5.20.62:8001

3.启动后zookeeper数据

		[zk: 33.33.33.101(CONNECTED) 2] ls /storm
		[workerbeats, errors, supervisors, storms, assignments]
		[zk: 33.33.33.101(CONNECTED) 3] ls /storm/supervisors
		[a36b8f0f-be1f-4f00-8a9b-6d15c3c81c41, c89242e1-74f0-4dbc-8777-d3bc0def950b]
		[zk: 33.33.33.101(CONNECTED) 4] ls /storm/assignments
		[]

#####kafka使用

添加下面内容到`/etc/security/limits.conf`

	kafka    -    nofile    98304

`server.properties`

	auto.create.topics.enable=true
已启动zookeeper集群的情况下：

	kafka-server-start.sh config/server.properties &
	kafka-server-start.sh config/server1.properties &
	kafka-server-start.sh config/server2.properties &
多个机器的server.properties

	brokerid=1
	port=9092
	log.dir=/home/vagrant/data/kafka/kafka-logs

清除Kafka里的所有数据
	
	rm -rf /home/vagrant/data/kafka/kafka-logs/*


#####kafka与storm配合使用注意
The maximum parallelism you can have on a KafkaSpout is the number of partitions of the corresponding Kafka topic
#####启动storm-starter
	$ storm jar storm-starter-*-jar-with-dependencies.jar storm.starter.RollingTopWords production-topology remote


SlidingWindowCounter

window length 3
窗口数组

 0 | 1 2 0 | 1 2 0 1 2

|headSlot|tailSlot|
|:------:|:------:|
|0       | 1      |
|1       |    2|
|2       |    0|
|0        |   1|
|1        |   2|
|2        |   0|

#####用户浏览数据库
1.表数据
	
	+-------+------------+------+-----+---------+----------------+
	| Field | Type       | Null | Key | Default | Extra          |
	+-------+------------+------+-----+---------+----------------+
	| id    | int(11)    | NO   | PRI | NULL    | auto_increment | 
	| uid   | int(11)    | NO   | MUL | NULL    |                | 
	| iid   | int(11)    | NO   |     | NULL    |                | 
	| lid   | int(11)    | NO   |     | 0       |                | 
	| aid   | int(11)    | NO   |     | 0       |                | 
	| ct    | bigint(20) | NO   |     | 0       |                | 
	| lvt   | int(11)    | NO   |     | 0       |                | 
	| done  | int(11)    | NO   |     | 0       |                | 
	| info  | text       | YES  |     | NULL    |                | 
	+-------+------------+------+-----+---------+----------------+

	1       374621267       170827220       0       0       1404662628968   4       0       {}
	2       374621267       194277413       0       0       1404662464718   171     0       {}
	3       394474629       130567778       -3      171248  1404662400001   827     0       {"device":2}
	4       403051945       130613823       -3      173364  1404662400003   212     0       {"device":2}
	5       397752039       130392336       -3      93801   1404662400015   1002    0       {"device":2}
	6       403044671       130935189       -3      96056   1404662400029   1438    0       {"device":2}
	7       401197008       130826307       -3      115971  1404662400016   962     0       {"device":2}
	8       393841073       130561186       -3      171007  1404662400011   533     0       {"device":2}
	9       391815506       131763989       -3      230799  1404662400023   629     0       {"device":2}
	10      401333949       130408838       -3      94995   1404662400079   2505    0       {"device":2}

导出测试数据集

	mysql -e "select * from ViewRecord_20140708 limit 10000" -uplaylistquality -ptdVrs-3n3w -h viewrecord-sdb1.jj.tudou.com -P 3307 viewrecord > ViewRecord_20140708_sample.sql

创建测试数据库


	CREATE TABLE `ViewRecord_20140707` (
	  `id` int(11) NOT NULL auto_increment,
	  `uid` int(11) default null,
	  `iid` int(11) default null,
	  `lid` int(11) NOT NULL default 0,
	  `aid` int(11) NOT NULL default 0,
	  `ct` bigint(20) NOT NULL default 0,
	  `lvt` int(11) NOT NULL default 0,
	  `done` int(11) NOT NULL default 0,
	  `info` text default null,
	  PRIMARY KEY  (`id`)
	)

建索引

	create index index_uid on ViewRecord_20140707(uid);

导入数据

	sqoop export --connect jdbc:mysql://localhost/viewrecord --username root --password 123456 --table ViewRecord_20140707  --export-dir /home/TagHierarchy/ViewRecord_20140707 --input-fields-terminated-by '\t' -m 18

日志设置:直接将下面的log4j.properties文件放到src/main/resource文件夹下，不用再添加如后面的配置代码

	log4j.rootLogger=DEBUG, CONSOLE
	#log4j.rootLogger=INFO, FILE
	
	log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
	log4j.appender.CONSOLE.Target=System.err
	log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
	log4j.appender.CONSOLE.layout.conversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %-5p - %m%n
	
	log4j.appender.FILE=org.apache.log4j.RollingFileAppender
	log4j.appender.FILE.File=log4j.log
	log4j.appender.FILE.MaxFileSize=512KB
	log4j.appender.FILE.MaxBackupIndex=3
	log4j.appender.FILE.layout=org.apache.log4j.PatternLayout
	log4j.appender.FILE.layout.conversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %-5p - %m%n



        Properties props = new Properties();
        try {
            props.load(MysqlToKafka.class
                    .getResourceAsStream("log4j.properties"));
        } catch (IOException e) {
            System.out.println(e.toString());
            log.error(e.toString());
        }
        PropertyConfigurator.configure(props);

配置端口转发用于测试

	Vagrant.configure("2") do |config|
	  config.vm.define :zktest1 do |zktest1|
	    zktest1.vm.provider "virtualbox" do |v|
	          v.customize ["modifyvm", :id, "--name", "zktest1", "--memory", "2048", "--cpus","2"]
	    end
	    zktest1.vm.box = "base"
	    zktest1.vm.hostname = "zktest1"
	    zktest1.vm.network :private_network, ip: "33.33.33.101"
	    zktest1.vm.network :forwarded_port, guest: 8080, host: 8080
	    zktest1.vm.network :forwarded_port, guest: 2181, host: 2182 
	  end
	
	  config.vm.define :zktest2 do |zktest2|
	    zktest2.vm.provider "virtualbox" do |v|
	          v.customize ["modifyvm", :id, "--name", "zktest2", "--memory", "2048", "--cpus","2"]
	    end
	    zktest2.vm.box = "base"
	    zktest2.vm.hostname = "zktest2"
	    zktest2.vm.network :private_network, ip: "33.33.33.102"
	    zktest2.vm.network :forwarded_port, guest: 2181, host: 2183 
	    zktest2.vm.network :forwarded_port, guest: 9092, host: 9093 
	  end
	
	  config.vm.define :zktest3 do |zktest3|
	    zktest3.vm.provider "virtualbox" do |v|
	          v.customize ["modifyvm", :id, "--name", "zktest3", "--memory", "2048", "--cpus","2"]
	    end
	    zktest3.vm.box = "base"
	    zktest3.vm.hostname = "zktest3"
	    zktest3.vm.network :private_network, ip: "33.33.33.103"
	    zktest3.vm.network :forwarded_port, guest: 2181, host: 2183 
	    zktest3.vm.network :forwarded_port, guest: 9092, host: 9094 
	  end
	end


从mysql中定时取viewrecord日志到Kafka

测试环境

	java -jar RtAnalysis-1.0-jar-with-dependencies.jar "zktest1:2181,zktest2:2181,zktest3:2181" viewrecord "33.33.33.103:9092,33.33.33.102:9092" jdbc:mysql://localhost:3306/viewrecord root 123456

生产环境

	java -jar RtAnalysis-1.0-jar-with-dependencies.jar "zktest1:2181,zktest2:2181,zktest3:2181" viewrecord "33.33.33.103:9092,33.33.33.102:9092" jdbc:mysql://viewrecord-sdb1.jj.tudou.com:3307/viewrecord playlistquality tdVrs-3n3w


#####数据持久化
1. Mongodb

#####总体的机器配置

	zktest1:zk,kafka,storm
	zktest2:zk,kafka,storm
	zktest3:zk,kafka,storm
	finger-test2:Mysql2Kafka
	finger-test2:mongodb

#####RTAnalysis启动
	storm jar RtAnalysis-1.0-jar-with-dependencies.jar com.ataosky.realtime.app.RollingTopVideos
**Reference**

1. [running-a-multi-broker-apache-kafka-cluster-on-a-single-node](http://www.michael-noll.com/blog/2013/03/13/running-a-multi-broker-apache-kafka-cluster-on-a-single-node/)

2. [kafka-storm-integration-example-tutorial](http://www.michael-noll.com/blog/2014/05/27/kafka-storm-integration-example-tutorial/)

3. [implementing-top-10-most-popular-articles-in-real-time-with-storm-and-mongodb](http://eugenedvorkin.com/implementing-top-10-most-popular-articles-in-real-time-with-storm-and-mongodb/)

4. [Storm starter源代码分析](http://www.cnblogs.com/fxjwind/archive/2013/05/22/3093018.html)

5. [implementing-real-time-trending-topics-in-storm](http://www.michael-noll.com/blog/2013/01/18/implementing-real-time-trending-topics-in-storm/)

6. [running-multi-node-storm-cluster](http://www.michael-noll.com/tutorials/running-multi-node-storm-cluster)




#####some
1.安装配置cdh4版本的hadoop
2.调试相似度计算的Hadoop程序，优化大量数据情况下的hadoop配置；
3.分析开源项目datafu中关于会话分析的部分；
#####一些并发的笔记
1. CountDownLatch与CyclicBarrier:
	
	CountDownLatch到达0后不能重用；不能用于等待并行线程的结束；


	CountDownLatch can be used to monitor the completion of the Children Threads if the size of the created children is known forehand. CountDownLatch enables a Thread or Threads to wait for completion of Children Threads. But there is no waiting amongst the Children until they finish each others tasks. Children may execute asynchronously and after their work is done will exit making a countdown. 


	CyclicBarrier can be used to create a set of Children Threads if the size of the Threads created is known forehand. CyclicBarrier can be used to implement waiting amongst Children Threads until all of them finish. This is useful where parallel threads needs to perform a job which requires sequential execution. For example 10 Threads doing steps 1, 2, 3, but all 10 Threads should finish step one before any can do step 2. Cyclic barrier can be reset after all Threads are finished execution. This is a distinguishing feature from a CountDownLatch. A CountDownLatch can only be used for a single count down. Additionally a CyclicBarrier can be assigned an Additional Thread which executes each time all the Children Threads finish their respective tasks. 

	Semaphore can be used to create a set of Children Threads even when the size of the Threads to be created is not known fore hand. This is because a Semaphore can wait until a number of releases have been made but that number is not required to initialize the Semaphore. Semaphores can be used in other scenarios such as Synchronizing between different threads such as Publisher, Subscriber scenario. 


