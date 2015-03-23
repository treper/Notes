import org.apache.spark.mllib.feature.Word2Vec
import java.io.{PrintWriter, FileOutputStream}

val input = sc.textFile("hdfs://yhd-jqhadoop2.int.yihaodian.com:8022/user/pms/xq/corpus_seg.txt").map(line => line.split(" ").toSeq)
val word2vec = new Word2Vec()
val model = word2vec.fit(input)
val vectors = model.getVectors

val out = new PrintWriter(new FileOutputStream("word_vectors.txt"))
vectors.foreach(line => out.write(m._1+"\t"+m._2.mkString(" ")+"\n"))
out.close()