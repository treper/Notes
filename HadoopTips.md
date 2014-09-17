####Hadoop Tips

The symptom here is that your code in your reduce phase is "stuck", either because of an infinite loop or just a ludicrous amount of data received, or something else (maybe post your reduce code?).

Here are the way that percentages work in the reducer:

0-33% is the shuffle. This is data moving from the mappers to the reducers (see how it starts before the mappers are finished).
33%-67% is the sort. This can only start when the mappers are finished (see how it goes from 30% to 67% after map is at 100%).
67%-100% is the actual reduce code you are running. This percentage goes up every time a reduce task completes. None of your reduce tasks are completing.
In the JobTracker interface, look at your job and see how much data the reducers are getting in. If the number of records in the reducer is going up, that means you probably have too much data going to the reducers. If that number stays still, you might have an infinite loop of some sort.


####Hadoop中JVM堆优化汇总及JVM复用

在mapred-site.xml配置文件里面有个mapred.child.java.opts配置，专门来配置一些诸如堆、垃圾回收之类的。


上面的mapred.task.java.opts属性是我们自己定义的，可以公布给用户配置；然后在mapred.child.java.opts中获取到mapred.task.java.opts的值，同时mapred.child.java.opts属性的final被设置为true，也就是不让客户修改。所以用户对mapred.child.java.opts直接配置是无效的；而且这里我们在获取${mapred.task.java.opts}之后再添加了-Xmx1000m，而在Java中，如果相同的jvm arg写在一起，比如”-Xmx2000m -Xmx1000m”，后面的会覆盖前面的，也就是说最终“-Xmx1000m”才会生效，通过这种方式，我们就可以有限度的控制客户端那边的heap size了。同样的道理，其他想覆盖的参数我们也可以写到后面。

