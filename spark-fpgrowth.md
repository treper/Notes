####spark PFPgrowth关联分析调优经验

* 数据用int替代string

70G压缩到10G左右

* 分组后数据先reduce，避免fpgrowth树循环次数太大，提前减少运算量

		
		(0 2 4 5 6 7 8 9 12 18 32 49 63 184,1112), (2 6 7 13 36 39 116 146 185,3), (1 2 7 10 13 38 46 151 183,1), (2 10 13 25 32 46 63 113 183,7), (0 2 10 25 32 35 53 74 75 182,1), (2 7 10 13 33 51 66 111 183,3), (1 2 3 6 13 31 35 40 88 182,19), (1 2 3 4 5 6 12 13 15 23 24 31 47 184,18), (0 2 3 4 5 6 12 15 20 21 23 33 53 185,84), (2 3 6 13 27 41 66 105 183,1), (0 2 3 6 17 42 46 181,99), (1 1 4 5 13 20 21 22 25 29 43 44 45 184,2), (1 2 6 13 25 43 66 158 183,2), (1 3 13 24 28 29 66 70 184,3), (0 2 6 25 34 39 56 103 104 180,3), (0 2 6 7 31 35 51 154 183,2), (0 2 3 6 36 81 185,1), (0 1 3 29 38 46 57 122 183,1), (0 2 3 6 17 42 46 52 142 181,75), (1 2 3 6 13 19 39 52 181,36), (1 4 13 24 25 29 34 50 57 181,2), (0 2 3 6 43 66 117 129 183,3), (0 2 6 7 33 35 40 86 97 181,101), (0 1 2 7 10 34 39 40 184,42)

分组后数据由200G压缩到1G左右

(0 2 4 5 6 7 8 9 12 18 32 49 63 184) x 1112

        for (record <- data) {
          for (item <- record) {
            if (!map.contains(item)) {
              val node: TreeNode = new TreeNode(item)
              node.count = 1
              map(item) = node
            } else {
              map(item).count += 1
            }
          }
        


        for (record <- data) {
          for (item <- record) {
            if (!map.contains(item)) {
              val node: TreeNode = new TreeNode(item)
              node.count = cnt//1112
              map(item) = node
            } else {
              map(item).count += cnt
            }
          }

* 大数据集不用GroupByKey,没有本地combine，shuffle消耗太大，用reduceByKey，combineByKey,foldByKey

![](http://databricks.gitbooks.io/databricks-spark-knowledge-base/content/images/group_by.png)

![](http://databricks.gitbooks.io/databricks-spark-knowledge-base/content/images/reduce_by.png)






* num-executor和executor-core的调整

		--executor-cores 8 --num-executors 3
		--executor-cores 2 -num-executors 12  lower gc,faster

* 分组数越大速度越快，但是相应的分组时间增加

		//data set size :10000  took 5.629 seconds.
		//data set size :110000 took 151.805 seconds.
		//data set size :3403606 took 8157.69 seconds.

* persist和checkpoint，步骤出错避免全部重新计算


		persist(StorageLevel.DISK_ONLY)

* sparkContext.set("spark.shuffle.consolidateFiles", "true")


[https://github.com/JerryLead/SparkInternals/blob/master/markdown/4-shuffleDetails.md](https://github.com/JerryLead/SparkInternals/blob/master/markdown/4-shuffleDetails.md)


* sparkContext.set("spark.default.parallelism", "300")

300个partition,executor-cores*num-executor个cpu并行计算

* JAVA_OPTS=" -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps"



* 序列化

kryoserializer



最新的测试结果

数据集

	5亿条交易记录

参数 

	--num-executors 15 --executor-cores 8 --executor-memory 20G

支持度2000，耗时96分钟,频繁项3653447


支持度200，耗时70分钟，频繁项15603604，提交参数

	/home/pms/spark-1.2.0-cdh5.02/bin/spark-submit --class fpm.example.PFPGrowthTest --master yarn-client --verbose --num-executors 10 --executor-cores 8 --executor-memory 20G --queue pms spark-test-1.0-SNAPSHOT-jar-with-dependencies.jar 0.0000004 " " 1500 "hdfs://yhd-jqhadoop2.int.yihaodian.com:8022/user/pms/xq/pms_user_data_prefix" "hdfs://yhd-jqhadoop2.int.yihaodian.com:8022/user/pms/xq/fpgrowth_result_1500_new" "4 6 7 8 9 11 14 15 16 17" "exf" 300


**Reference**

[http://rdc.taobao.org/?p=533](http://rdc.taobao.org/?p=533)



spark版本关联分析对5亿条购买数据（70G），从18个维度中选择了10个维度进行关联分析，将字符串编码为整型数后分1000个分组，支持度设置为2000，挖掘到所有满足支持度的**多元**频繁项集3,653,447条，耗时96分钟。

hadoop版本的算法将数据分5000组，耗时差不多1.5小时,取频繁模式的top K（目前取80），得到**二元,三元**频繁项集100多万条，丢失了部分挖掘到的频繁项集，由于多元的频繁项集结果没有所以无法做具体的速度对比。


spark-submit-onyarn --class fpm.example.PFPGrowthTest --master yarn-client --verbose --num-executors 4 --executor-cores 16 --executor-memory 20G --queue pms spark-test-1.0-SNAPSHOT-jar-with-dependencies.jar 0.0000004 " " 1500 "hdfs://yhd-jqhadoop2.int.yihaodian.com:8022/user/pms/xq/pms_user_data_prefix_space" "hdfs://yhd-jqhadoop2.int.yihaodian.com:8022/user/pms/xq/fpgrowth_result_1500" "4 6 7 8 9 11 14 15 16 17" "exf" 300

spark-shell-onyarn --num-executors 2 --executor-cores 12 --executor-memory 20G --queue pms
