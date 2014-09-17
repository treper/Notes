shuffle过程
 

map端：

    Map 开始产生输出时，它并不是简单的把数据写到磁盘，因为频繁的磁盘操作会导致性能严重下降。它的处理过程更复杂，数据首先是写到内存中的一个缓冲区，并做了一些预排序，以提升效率。

    每个Map 任务都有一个用来写入输出数据的循环内存缓冲区。这个缓冲区默认大小是100MB，可以通过io.sort.mb 属性来设置具体大小。当缓冲区中的数据量达到一个特定阀值(io.sort.mb * io.sort.spill.percent，其中io.sort.spill.percent 默认是0.80)时，系统将会启动一个后台线程把缓冲区中的内容spill 到磁盘。在spill 过程中，Map 的输出将会继续写入到缓冲区，但如果缓冲区已满，Map 就会被阻塞直到spill 完成。spill 线程在把缓冲区的数据写到磁盘前，会对它进行一个二次快速排序，首先根据数据所属的partition 排序，然后每个partition 中再按Key 排序。输出包括一个索引文件和数据文件。如果设定了Combiner，将在排序输出的基础上运行。Combiner 就是一个Mini Reducer，它在执行Map 任务的节点本身运行，先对Map 的输出做一次简单Reduce，使得Map 的输出更紧凑，更少的数据会被写入磁盘和传送到Reducer。spill 文件保存在由mapred.local.dir指定的目录中，Map 任务结束后删除。

    每当内存中的数据达到spill 阀值的时候，都会产生一个新的spill 文件，所以在Map任务写完它的最后一个输出记录时，可能会有多个spill 文件。在Map 任务完成前，所有的spill 文件将会被归并排序为一个索引文件和数据文件，如图3 所示。这是一个多路归并过程，最大归并路数由io.sort.factor 控制(默认是10)。如果设定了Combiner，并且spill文件的数量至少是3（由min.num.spills.for.combine 属性控制），那么Combiner 将在输出文件被写入磁盘前运行以压缩数据

reduce端：

    Shuffle在reduce端主要分为两个阶段即复制阶段（copy phase）和排序阶段（sort phase 合并阶段更合适点，排序在map端进行）。也就是说Reducer真正运行之前，所有的时间都是在拉取数据，做merge，且不断重复地在做。 
    1.Copy过程，简单地拉取数据。Reduce进程启动一些数据copy线程(Fetcher)，通过HTTP方式请求map task所在的TaskTracker获取map task的输出文件。因为map task早已结束，这些文件就归TaskTracker管理在本地磁盘中。 

    2.Merge阶段。这里的merge如map端的merge动作，只是数组中存放的是不同map端copy来的数值。Copy过来的数据会先放入内存缓冲区中，这里的缓冲区大小要比map端的更为灵活，它基于JVM的heap size设置，因为Shuffle阶段Reducer不运行，所以应该把绝大部分的内存都给Shuffle用。这里需要强调的是，merge有三种形式：1)内存到内存  2)内存到磁盘  3)磁盘到磁盘。默认情况下第一种形式不启用，让人比较困惑，是吧。当内存中的数据量到达一定阈值，就启动内存到磁盘的merge。第二种merge方式一直在运行，直到没有map端的数据时才结束，然后启动第三种磁盘到磁盘的merge方式生成最终的那个文件。 