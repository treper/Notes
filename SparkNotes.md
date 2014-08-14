#####Windows下配置Spark开发环境

1. 安装scala

	a. 下载`scala-2.11.1.msi`并安装，设置`SCALA_HOME=C:\Program Files\scala`并添加到`Path`系统变量

2. 配置intellij idea

	a. 已安装scala和sbt的plugin


3. Spark-shell测试

		MASTER=local[24] ./spark-shell


**Reference**

[配置SCALA](http://lancegatlin.org/tech/intellij_idea-configure-for-scala-and-sbt)

[视频](http://www.scalacourses.com/student/showLecture/80)

[http://cn.soulmachine.me/blog/20140130/](http://cn.soulmachine.me/blog/20140130/)