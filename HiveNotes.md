#####Hive常用操作
**建表**
	

	create table md5id_vv(md5id int,vv int)
	row format delimited
	fields terminated by '\t'
	stored as textfile;


其他常用类型biging,string

导入本地数据

	load data local inpath "/home/mps/software/tag_score_sort" into table tag_score;

导入hdfs数据


 select sum(vv) from md5id_vv;

**表描述**

	show tables;
	desc table;

**常用查询**
	
	select distinct(tagid) from labeldic_type3;
	select max(*) from table;  
	select sum(*) from table;   

**UDF**

* UDF：操作单个数据行，产生单个数据行；
* UDAF：操作多个数据行，产生一个数据行。
* UDTF：操作一个数据行，产生多个数据行一个表作为输出。

		add jar testudf.jar
		create temporary function md5tagcount as 'com.ataosky.hive.udtf.MD5TagMapCount';  
		create table md5idlist (md5id int,list string) 
		row format delimited
		fields terminated by '\t'
		stored as textfile;
		
		load data local inpath "/home/mps/software/md5_labels_sample1k" into table md5idlist;
		
		select md5tagcount(list) as (word, cnt) from md5idlist;

可作为map,reduce逻辑插入到hive sql中

#####优化

hive.optimize.cp=true：列裁剪
hive.optimize.prunner：分区裁剪
hive.limit.optimize.enable=true：优化LIMIT n语句
hive.limit.row.max.size=1000000：
hive.limit.optimize.limit.file=10：最大文件数
1. 本地模式(小任务)：
需要满足以下条件：
　　1.job的输入数据大小必须小于参数：hive.exec.mode.local.auto.inputbytes.max(默认128MB)
　　2.job的map数必须小于参数：hive.exec.mode.local.auto.tasks.max(默认4)
　　3.job的reduce数必须为0或者1
hive.exec.mode.local.auto.inputbytes.max=134217728
hive.exec.mode.local.auto.tasks.max=4
hive.exec.mode.local.auto=true
hive.mapred.local.mem：本地模式启动的JVM内存大小
2. 并发执行：
hive.exec.parallel=true ，默认为false
hive.exec.parallel.thread.number=8
3.Strict Mode：
hive.mapred.mode=true，严格模式不允许执行以下查询：
分区表上没有指定了分区
没有limit限制的order by语句
笛卡尔积：JOIN时没有ON语句
4.动态分区：
hive.exec.dynamic.partition.mode=strict：该模式下必须指定一个静态分区
hive.exec.max.dynamic.partitions=1000
hive.exec.max.dynamic.partitions.pernode=100：在每一个mapper/reducer节点允许创建的最大分区数
DATANODE：dfs.datanode.max.xceivers=8192：允许DATANODE打开多少个文件
5.Single MapReduce MultiGROUP BY
hive.multigroupby.singlemar=true：当多个GROUP BY语句有相同的分组列，则会优化为一个MR任务
6. hive.exec.rowoffset：是否提供虚拟列
7. 分组
两个聚集函数不能有不同的DISTINCT列，以下表达式是错误的：
INSERT OVERWRITE TABLE pv_gender_agg SELECT pv_users.gender, count(DISTINCT pv_users.userid), count(DISTINCT pv_users.ip) FROM pv_users GROUP BY pv_users.gender;
SELECT语句中只能有GROUP BY的列或者聚集函数。
8.
hive.map.aggr=true;在map中会做部分聚集操作，效率更高但需要更多的内存。
hive.groupby.mapaggr.checkinterval：在Map端进行聚合操作的条目数目
9
hive.groupby.skewindata=true：数据倾斜时负载均衡，当选项设定为true，生成的查询计划会有两个MRJob。第一个MRJob 中，
Map的输出结果集合会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的GroupBy Key
有可能被分发到不同的Reduce中，从而达到负载均衡的目的；第二个MRJob再根据预处理的数据结果按照GroupBy Key分布到
Reduce中（这个过程可以保证相同的GroupBy Key被分布到同一个Reduce中），最后完成最终的聚合操作。
10.Multi-Group-By Inserts：
FROM test
INSERT OVERWRITE TABLE count1
SELECT count(DISTINCT test.dqcode)
GROUP BY test.zipcode
INSERT OVERWRITE TABLE count2
SELECT count(DISTINCT test.dqcode)
GROUP BY test.sfcode;
11.排序
ORDER BY colName ASC/DESC
hive.mapred.mode=strict时需要跟limit子句
hive.mapred.mode=nonstrict时使用单个reduce完成排序
SORT BY colName ASC/DESC ：每个reduce内排序
DISTRIBUTE BY(子查询情况下使用 )：控制特定行应该到哪个reducer，并不保证reduce内数据的顺序
CLUSTER BY ：当SORT BY 、DISTRIBUTE BY使用相同的列时。
 

12.合并小文件
hive.merg.mapfiles=true：合并map输出
hive.merge.mapredfiles=false：合并reduce输出
hive.merge.size.per.task=256*1000*1000：合并文件的大小
hive.mergejob.maponly=true：如果支持CombineHiveInputFormat则生成只有Map的任务执行merge
hive.merge.smallfiles.avgsize=16000000：文件的平均大小小于该值时，会启动一个MR任务执行merge。
 

13.map/reduce数目
减少map数目：
　　set mapred.max.split.size
　　set mapred.min.split.size
　　set mapred.min.split.size.per.node
　　set mapred.min.split.size.per.rack
　　set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat
增加map数目：
当input的文件都很大，任务逻辑复杂，map执行非常慢的时候，可以考虑增加Map数，来使得每个map处理的数据量减少，从而提高任务的执行效率。
假设有这样一个任务：
　　select data_desc, count(1), count(distinct id),sum(case when …),sum(case when ...),sum(…) from a group by data_desc
如果表a只有一个文件，大小为120M，但包含几千万的记录，如果用1个map去完成这个任务，肯定是比较耗时的，这种情况下，我们要考虑将这一个文件合理的拆分成多个，这样就可以用多个map任务去完成。
　　set mapred.reduce.tasks=10;
　　create table a_1 as select * from a distribute by rand(123);
这样会将a表的记录，随机的分散到包含10个文件的a_1表中，再用a_1代替上面sql中的a表，则会用10个map任务去完成。每个map任务处理大于12M（几百万记录）的数据，效率肯定会好很多。
reduce数目设置：
　参数1：hive.exec.reducers.bytes.per.reducer=1G：每个reduce任务处理的数据量
　参数2：hive.exec.reducers.max=999(0.95*TaskTracker数)：每个任务最大的reduce数目
　reducer数=min(参数2,总输入数据量/参数1)
　set mapred.reduce.tasks：每个任务默认的reduce数目。典型为0.99*reduce槽数，hive将其设置为-1，自动确定reduce数目。
 

14.使用索引：
hive.optimize.index.filter：自动使用索引
hive.optimize.index.groupby：使用聚合索引优化GROUP BY操作


#####日志导入

1. 建表

		create table(id INT,uid INT,iid INT,lid INT,aid INT,ct INT,lvt INT,done INT,info STRING) PARTITIONED BY (recordDate INT) CLUSTERED BY(uid) SORTED BY(lvt) INTO 16 BUCKETS ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';




hive> create table ViewRecord(id INT,uid INT,iid INT,lid INT,aid INT,ct INT,lvt INT,done INT,info STRING) PARTITIONED BY (recordDate INT) CLUSTERED BY(uid) SORTED BY(lvt) INTO 16 BUCKETS ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';  
OK
Time taken: 3.981 seconds
hive> set hive.enforce.bucketing = true;
hive> LOAD DATA LOCAL INPATH 'ViewRecord_20140824' OVERWRITE INTO TABLE ViewRecord;                              
FAILED: SemanticException [Error 10062]: Need to specify partition columns because the destination table is partitioned
hive> LOAD DATA LOCAL INPATH 'ViewRecord_20140824' OVERWRITE INTO TABLE ViewRecord PARTITION (recordDate=20140824);
Copying data from file:/home/mps/software/ViewRecord_20140824
Copying file: file:/home/mps/software/ViewRecord_20140824
Loading data to table default.viewrecord partition (recorddate=20140824)
Partition default.viewrecord{recorddate=20140824} stats: [num_files: 1, num_rows: 0, total_size: 3067528656, raw_data_size: 0]
Table default.viewrecord stats: [num_partitions: 1, num_files: 1, num_rows: 0, total_size: 3067528656, raw_data_size: 0]
OK
Time taken: 36.587 seconds
hive> 

FAILED: ParseException line 1:12 extraneous input '(' expecting Identifier near 'id' in create table statement

hive是不支持insert into 语句的
分片分桶对每天的日志
hive不支持更新，事务和索引

#####分区和桶

分区的好处是对于同一分区内的数据进行查询的时候将会非常高效，比如按年份分区，查某一年的数据只需要到这个分区中检索就行了。分区是在创建表的时候用 PARTITIONED BY 子句定义的。

	CREATE TABLE LOGS(ts BEGINT,line STRING)

	PARTITIONED BY (dt STRING,country STRING);

在LOAD数据的时候则需要指定分区值：

	LOAD DATA LOCAL INPUT 'input/hive/partitions/file1'

    INTO TABLE logs

    PARTITION (dt='2001-1-1',country='GB');
       
在文件系统中，分区只是表目录下嵌套的子目录。如按分区检索数据其实就是按文件系统目录检索数据了。

SHOW PARTITIONS logs; 命令会返回当前表中所有分区信息。PARTITIONED BY 子句中的列是表中正式的列，称为分区列。但是数据文件并不包含这些列，因为在目录中包含了。可以像普通列一样使用分区列（partition column）

	select ts,dt,line
	  from logs
	 where country='GB'

按照JOIN顺序中的最后一个表应该尽量是大表




hive -e "select md5_labels.lables,lis_video_rank_d1_20140501.vv from md5_labels join md5Id_md5 on (md5_labels.md5id=md5Id_md5.md5id) join itemId_md5 on (md5Id_md5.md5=itemId_md5.md5) join lis_video_rank_d1_20140501 on (itemId_md5.itemid=lis_video_rank_d1_20140501.itemid);" > labels_vv
hadoop jar labelsVV.jar /home/TagHierarchy/labels_vv /home/TagHierarchy/label_vv
hadoop fs -getmerge /home/TagHierarchy/label_vv label_vv
hive -e "select label_vv.vv,labeldic_type3.tag from labeldic_type3 join label_vv on (label_vv.labelid=labeldic_type3.tagid) order by label_vv.vv desc;" > tagtype3_vv_sort

hive -e "select md5_labels.lables,lis_video_rank_d1_20140501.score from md5_labels join md5Id_md5 on (md5_labels.md5id=md5Id_md5.md5id) join itemId_md5 on (md5Id_md5.md5=itemId_md5.md5) join lis_video_rank_d1_20140501 on (itemId_md5.itemid=lis_video_rank_d1_20140501.itemid);" > labels_score
hadoop jar LabelsScore.jar /home/TagHierarchy/labels_score /home/TagHierarchy/tagId_score
hive -e "select tag_score.score,labeldic_type3.tag from tag_score join labeldic_type3 on (tag_score.tag=labeldic_type3.tagid) order by tag_score.score desc;" > tagtype3_score_sort

Strange hadoop Problem,I am using custom Writeable Object VectorComponentArrayWritable according to the name, It is a vector.The input data is sequencially processed by job A and B.Job A's output is set to SequenceFileOutputFormat and input format of job B's  is set to SequenceFileInputFormat. When I use 10,000 records or 1,000,000 records as Input.The output of Job A is SequenceFile and it can be processed successfully by job B,whose input is set to SeqenceFileInputFormat without error.When I use 40,000,000 records as input,the job complete successfully .But I found the output is not a SequenceFile as job B throws following error:

[mps@finger-test1 bigdata]$ hadoop jar simjoin-1.0.jar aps.Cdh4MaxWi /mps/md5_labels_processed/part-r-00000 /mps/md5_labels_maxwi_out
14/08/01 11:29:53 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
14/08/01 11:29:54 INFO service.AbstractService: Service:org.apache.hadoop.yarn.client.YarnClientImpl is inited.
14/08/01 11:29:54 INFO service.AbstractService: Service:org.apache.hadoop.yarn.client.YarnClientImpl is started.
14/08/01 11:29:54 WARN mapreduce.JobSubmitter: Use GenericOptionsParser for parsing the arguments. Applications should implement Tool for the same.
14/08/01 11:29:54 INFO input.FileInputFormat: Total input paths to process : 1
14/08/01 11:29:54 INFO mapreduce.JobSubmitter: number of splits:168
14/08/01 11:29:54 WARN conf.Configuration: mapred.jar is deprecated. Instead, use mapreduce.job.jar
14/08/01 11:29:54 WARN conf.Configuration: mapred.output.value.class is deprecated. Instead, use mapreduce.job.output.value.class
14/08/01 11:29:54 WARN conf.Configuration: mapreduce.combine.class is deprecated. Instead, use mapreduce.job.combine.class
14/08/01 11:29:54 WARN conf.Configuration: mapreduce.map.class is deprecated. Instead, use mapreduce.job.map.class
14/08/01 11:29:54 WARN conf.Configuration: mapred.job.name is deprecated. Instead, use mapreduce.job.name
14/08/01 11:29:54 WARN conf.Configuration: mapreduce.reduce.class is deprecated. Instead, use mapreduce.job.reduce.class
14/08/01 11:29:54 WARN conf.Configuration: mapreduce.inputformat.class is deprecated. Instead, use mapreduce.job.inputformat.class
14/08/01 11:29:54 WARN conf.Configuration: mapred.input.dir is deprecated. Instead, use mapreduce.input.fileinputformat.inputdir
14/08/01 11:29:54 WARN conf.Configuration: mapred.output.dir is deprecated. Instead, use mapreduce.output.fileoutputformat.outputdir
14/08/01 11:29:54 WARN conf.Configuration: mapreduce.outputformat.class is deprecated. Instead, use mapreduce.job.outputformat.class
14/08/01 11:29:54 WARN conf.Configuration: mapred.map.tasks is deprecated. Instead, use mapreduce.job.maps
14/08/01 11:29:54 WARN conf.Configuration: mapred.output.key.class is deprecated. Instead, use mapreduce.job.output.key.class
14/08/01 11:29:54 WARN conf.Configuration: mapred.working.dir is deprecated. Instead, use mapreduce.job.working.dir
14/08/01 11:29:54 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1406199511149_0018
14/08/01 11:29:54 INFO client.YarnClientImpl: Submitted application application_1406199511149_0018 to ResourceManager at finger-test1/10.25.13.151:8032
14/08/01 11:29:54 INFO mapreduce.Job: The url to track the job: http://finger-test1:8088/proxy/application_1406199511149_0018/
14/08/01 11:29:54 INFO mapreduce.Job: Running job: job_1406199511149_0018
14/08/01 11:29:59 INFO mapreduce.Job: Job job_1406199511149_0018 running in uber mode : false
14/08/01 11:29:59 INFO mapreduce.Job:  map 0% reduce 0%
14/08/01 11:30:02 INFO mapreduce.Job: Task Id : attempt_1406199511149_0018_m_000000_0, Status : FAILED
Error: java.io.IOException: hdfs://finger-test1:9100/mps/md5_labels_processed/part-r-00000 not a SequenceFile
        at org.apache.hadoop.io.SequenceFile$Reader.init(SequenceFile.java:1805)
        at org.apache.hadoop.io.SequenceFile$Reader.initialize(SequenceFile.java:1765)
        at org.apache.hadoop.io.SequenceFile$Reader.<init>(SequenceFile.java:1714)
        at org.apache.hadoop.io.SequenceFile$Reader.<init>(SequenceFile.java:1728)
        at org.apache.hadoop.mapreduce.lib.input.SequenceFileRecordReader.initialize(SequenceFileRecordReader.java:54)
        at org.apache.hadoop.mapred.MapTask$NewTrackingRecordReader.initialize(MapTask.java:518)
        at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:755)
        at org.apache.hadoop.mapred.MapTask.run(MapTask.java:338)
        at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:160)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:415)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1438)
        at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:155)

and I have inspceted the output it is TextOutputFormat. I don't know why this error occurs because this is really strange. Have anybody encountered this kind of problem? 