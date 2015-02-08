####Spark 1.2编译

shell:

export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=512m"

mvn -Pyarn -Phadoop-2.3 -Dhadoop.version=2.3.0-cdh5.0.2 -Phive -Phive-thriftserver -DskipTests clean package