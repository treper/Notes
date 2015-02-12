####Spark优化

####Spark SQL Notes

#####UDF

使用Java开发一个helloworld级别UDF，打包成udf.jar，存放在/home/Hadoop/lib下，代码如下：

package com.luogankun.udf;
import org.apache.hadoop.hive.ql.exec.UDF;
public class HelloUDF extends UDF {
    public String evaluate(String str) {
        try {
            return "HelloWorld " + str;
        } catch (Exception e) {
            return null;
        }
    }
}  
Hive中使用UDF

cd $SPARK_HOME/bin
spark-sql --jars /home/hadoop/lib/udf.jar
CREATE TEMPORARY FUNCTION hello AS 'com.luogankun.udf.HelloUDF';
select hello(url) from page_views limit 1;  
SparkSQL中使用UDF

方式一：在启动spark-sql时通过--jars指定

cd $SPARK_HOME/bin
spark-sql --jars /home/hadoop/lib/udf.jar
CREATE TEMPORARY FUNCTION hello AS 'com.luogankun.udf.HelloUDF';select hello(url) from page_views limit 1; 
方式二：先启动spark-sql后add jar

cd $SPARK_HOME/bin
spark-sql
add jar /home/hadoop/lib/udf.jar;
CREATE TEMPORARY FUNCTION hello AS 'com.luogankun.udf.HelloUDF';
select hello(url) from page_views limit 1; 
在测试过程中发现并不支持该种方式，会报java.lang.ClassNotFoundException: com.luogankun.udf.HelloUDF

如何解决？ 

1）需要先将udf.jar的路径配置到spark-env.sh的SPARK_CLASSPATH中，形如：

export SPARK_CLASSPATH=$SPARK_CLASSPATH:/home/hadoop/software/mysql-connector-java-5.1.27-bin.jar:/home/hadoop/lib/udf.jar 
2）再启动spark-sql，直接CREATE TEMPORARY FUNCTION即可；

cd $SPARK_HOME/bin
spark-sql
CREATE TEMPORARY FUNCTION hello AS 'com.luogankun.udf.HelloUDF';
select hello(url) from page_views limit 1; 
方式三：Thrift JDBC Server中使用UDF

在beeline命令行中执行：

add jar /home/hadoop/lib/udf.jar;
CREATE TEMPORARY FUNCTION hello AS 'com.luogankun.udf.HelloUDF';
select hello(url) from page_views limit 1;




Spark SQL中缓存表一定要用cacheTable(“tableName”)这种形式，否则无法享受到列式存储带来的一系列好处，但是很多朋友仍然采用rdd.cache这种原生的方式来缓存，社区也意识到这样不行，所以现在无论是cacheTable还是直接cache，都是表达相同的语义，都能享受到列式存储带来的好处。

