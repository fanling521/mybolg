# Hadoop-Yarn

Yarn是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于运行于操作系统之上的应用程序。

## Yarn的基本架构

（1）**ResourceManager**处理客户端的请求，监控NodeManager，启动或监控ApplicationMaster，负责资源的分配和调度，由调度和ApplicationsMaster两个组件。

（2）**NodeManager**管理单个节点上的资源，处理来自ResourceManager和ApplicationMaster的命令

（3）**ApplicationMaster**负责数据的切割，为应用程序申请资源并分配给内部任务，负责任务的调度和容错

（4）**Container**是YARN上的资源抽象，封装了某个节点上多维度资源。

（5）**MapTask**

（6）**ReduceTask**

## Yarn的执行过程

![工作流程](assets/20190417203747.png)

1. Client向RM提交任务
2. RM通过心跳机制查看NM的资源使用情况，选定一个NM来分配Containner来启动AM
3. AM向RM注册，这样用户可以在RM上看到程序的运行状态，它为各个任务申请资源，监控运行状态，直到运行结束，其中
   1. AM通过RPC协议向RM申请和领取资源
   2. 申请到资源，和对应的NM通信，启动任务MRAppMaster
   3. NM准备好环境后，MRAppMaster启动MapTask或ReduceTask
   4. 各个任务通过PRC协议向AM汇报状态和进度
4. 应用程序结束后，AM向RM申请注销自己

### MapTask工作机制

1. **Read阶段**：MapTask通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。
2. **Map阶段**：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。
3. **Collect收集阶段**：在用户编写map()函数中，当数据处理完成后，一般会调用`OutputCollector.collect()`输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。
4. **Spill阶段**：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。

### ReduceTask工作机制

1. **Copy阶段**：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。
2. **Merge阶段**：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。
3. **Sort阶段**：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。
4. **Reduce阶段**：reduce()函数将计算结果写到HDFS上。

