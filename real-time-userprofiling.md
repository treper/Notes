#####KafkaSpout

topic名字
当前spout的唯一标识Id （以下代称$spout_id）
zookeeper上用于存储当前处理到哪个Offset了 （以下代称$zk_root)
当前topic中数据如何解码
了解Kafka的应该知道，Kafka中当前处理到哪的Offset是由客户端自己管理的。所以，后面两个的目的，其实是在zookeeper上建立一个 $zk_root/$spout_id 的节点，其值是一个map，存放了当前Spout处理的Offset的信息。