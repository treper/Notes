#####CentOS .57 Cloudera Manager安装CDH4.7

	sudo useradd -r --home=/home/mps/software/cm-4.8.1/run/cloudera-scm-server --comment "Cloudera SCM User" cloudera-scm


遇到问题

	[mps@finger-test2 cm-4.8.1]$ sudo ./share/cmf/schema/scm_prepare_database.sh mysql scm  -hlocalhost -uroot -p123456  --scm-host localhost scm scm scm
	+======================================================================+
	|      Error: JAVA_HOME is not set and Java could not be found         |
	+----------------------------------------------------------------------+
	| Please download the latest Oracle JDK from the Oracle Java web site  |
	|  > http://www.oracle.com/technetwork/java/javase/index.html <        |
	|                                                                      |
	| Cloudera Manager requires Java 1.6 or later.                         |
	| NOTE: This script will find Oracle Java whether you install using    |
	|       the binary or the RPM based installer.                         |
	+======================================================================+

解决：

	grep JAVA_HOME ./share/cmf/schema/scm_prepare_database.sh -nr

添加`/usr/local/jdk1.7*`

	sudo chown -R mps:mps cm-4.8.1
	sudo chown -R mps:mps cloudera
	./cm-4.8.1/etc/init.d/cloudera-scm-server start
	scp -r  cm-4.8.1 mps@finger-test1:/home/mps/software
	/home/mps/software/cm-4.8.1/etc/init.d/cloudera-scm-agent start



	grant all on *.* to root@"finger-test1" Identified by "123456" ; 
	grant all on *.* to root@"finger-test2" Identified by "123456" ; 

finger-test2上：

	./etc/init.d/cloudera-scm-server start
	./etc/init.d/cloudera-scm-agent start

finger-test1上：

	./etc/init.d/cloudera-scm-agent start

修改`Local Parcel Repository Path`为`/home/mps/software/cloudera/parcel-repo`并修改`update frequency`为1分钟


**Reference**:

[http://www.cnblogs.com/thinkCoding/p/3567408.html](http://www.cnblogs.com/thinkCoding/p/3567408.html)

[http://www.tuicool.com/articles/EfeEzu](http://www.tuicool.com/articles/EfeEzu)

[http://www.wangyongkui.com/hadoop-cdh5/](http://www.wangyongkui.com/hadoop-cdh5/)