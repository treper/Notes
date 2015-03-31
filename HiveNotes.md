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


添加行号

add jar /home/zhoushugang/apache-hive-0.13.1-bin/lib/hive-contrib-0.13.1.jar;
drop temporary function row_sequence;
create temporary function row_sequence as 'org.apache.hadoop.hive.contrib.udf.UDFRowSequence';