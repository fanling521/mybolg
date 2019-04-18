# Hadoop-YARN

Yarn是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于运行于操作系统之上的应用程序。

## YARN基本架构

### ResourceManager

- 处理客户端的请求
- 监控NodeManager
- 启动或监控ApplicationMaster
- 资源的分配和调度

### NodeManager

- 管理单个节点上的资源
- 处理来自ResourceManager的命令
- 处理来自ApplicationMaster的命令

### ApplicationMaster

- 负责数据的切割
- 为应用程序申请资源并分配给内部任务
- 任务的调度和容错

### Container

Container是YARN上的资源抽象，封装了某个节点上多维度资源。

## YARN 工作流程

![工作流程](assets/20190417203747.png)

1. 用户向RM提交了一个jar包任务
2. RM通过心跳机制查看NM的资源使用情况，选定一个NM来分配Containner来启动AM
3. AM向RM注册，这样用户可以在RM上看到程序的运行状态，它为各个任务申请资源，监控运行状态，直到运行结束，其中
   1. AM通过RPC协议向RM申请和领取资源
   2. 一旦申请到资源，和对应的NM通信，启动任务
   3. NM为任务准备好环境后，运行任务
   4. 各个任务通过PRC协议向AM汇报状态和进度
4. 应用程序结束后，AM向RM申请注销自己



