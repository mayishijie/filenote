# 1. flink整体架构以及重要特点

下面这张图是flink的流批统一以及事件驱动的重要概念图

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7921b9f6da14c47b3c38098088ec9d3~tplv-k3u1fbpfcp-watermark.image)

## 1. flink的特点

### 1 事件驱动型

> 事件驱动型运用是一类具有状态的应用，它从一个或者多个事件流中提取数据，并根据到来的事件触发计算，状态更新。类似于kafka这种消息队列都是事件驱动型应用。

事件驱动型概念图

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ff2c94daf5b4a1a880fd3556bf67f61~tplv-k3u1fbpfcp-watermark.image)

### 2. 流的世界观

> 1. 有界流：离线数据
> 2. 无界流：实时数据

### 3. 分层api

> 

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f005d49e0c54b22a1783b22744e748f~tplv-k3u1fbpfcp-watermark.image)

## 2. yarn模式下

### 1. session-cluster模式

> 在yarn中初始化一个集群，以后所有的flink job都会向这里提交，这个flink集群常驻在yarn中，除非手动停止（**一般是在测试环境中会这样，生成环境一般不会这样搞**）

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/056ef3bef70945c1abd94d8acbdbd52b~tplv-k3u1fbpfcp-watermark.image)

### 2. 启动

1. 启动Hadoop集群

2. 启动yarn-session

   ```shell
   ./yarn-session.sh -n 2 -s 2 -jm 1024 -tm 1024 -nm test -d
   ```

   名词解释：

   >-n(--container) : taskManager的数量
   >
   >-s(--slots): 每个taskManager的slot数量，默认一个slot一个core，
   >
   >-jm: jobManager的内存（MB）
   >
   >-tm： taskManager的内存（MB）
   >
   >-nm: yarn的appName(现在yarn的ui上的名字)。 
   >
   >-d: 后台执行

3. 执行任务

   ```shell
   ./flink run -c com.mayishijie.wc.StreamWordCount  FlinkTutorial-1.0-SNAPSHOT-jar-with-dependencies.jar --host lcoalhost –port 7777
   ```

4. 取消yarn-session

   ```shell
   yarn application --kill application_1577588252906_0001
   ```

   

### 2. per-job-cluster模式

> 每次提交都会创建一个新的flink集群，任务之间互相独立，互不影响，方便管理。任务执行完成之后创建的集群也会消失

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d03edf39d514e3dacee39f79730a924~tplv-k3u1fbpfcp-watermark.image)

### 启动

> 和session-cluster的步骤差不多，就是不用启动yarn-session,**直接运行任务**

# 2. Flink运行时架构

## 1. 运行时4大组件

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c2b0f9619694310b50a4163eb5d538d~tplv-k3u1fbpfcp-watermark.image)

## 2. 各个组件的职责

### 1. jobManger（控制一个应用程序执行的主进程）

### 2. TaskManger

### 3. ResourceManger

### 4. dispacher

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba0202a9f1e94e97a4a19dac7611212a~tplv-k3u1fbpfcp-watermark.image)

## 3. 提交流程

### 1. 大体流程

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/880b5ab32293417c829334b44f6a8bdb~tplv-k3u1fbpfcp-watermark.image)

### 2. Yarn下的提交流程详解

其实yarn下无论是flink还是其他，可以参考mr中在yarn下的提交详细流程，其实都差不多

> 1. Flink任务提交后，Client向HDFS上传Flink的Jar包和配置，
> 2. 之后向Yarn ResourceManager提交任务，ResourceManager分配Container资源并通知对应的NodeManager启动ApplicationMaster，
> 3. ApplicationMaster启动后加载Flink的Jar包和配置构建环境，然后启动JobManager，
> 4. 之后ApplicationMaster向ResourceManager申请资源启动TaskManager，ResourceManager分配Container资源后，由ApplicationMaster通知资源所在节点的NodeManager启动TaskManager
> 5. NodeManager加载Flink的Jar包和配置构建环境并启动TaskManager，TaskManager启动后向JobManager发送心跳包，并等待JobManager向其分配任务。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b02898b7c3542abb015cf95e701e15e~tplv-k3u1fbpfcp-watermark.image)

## 4. 任务调度原理

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bc0d89503e04c7dae05f95bc0add979~tplv-k3u1fbpfcp-watermark.image)

### 1. taskManager与slots

#### 1. taskManager与slots的关系图

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ace3dd0f7f5c4417a71483a48049ebd7~tplv-k3u1fbpfcp-watermark.image)

> 1. 一个worker(TaskManager)是一个JVM进程
> 2. worker能接收多少个task，取决于worker有多少个task slot
> 3. 一个task slot就是taskManager中的资源子集（**相当于就是有独立的一块资源集**），这也意味着它不用与其它job的subTask竞争管理的内存
> 4. slot目前仅仅是用来隔离task的受管理的内存
> 5. **重点理解6和7**：
> 6. **task slot是个静态概念**，是指taskManager的并发能力（其实就是它拥有多少的资源集，有多少task slot，就有多少的并发能力。可以通过**taskmanager.numberOfTaskSlots**配置）
> 7. **并行度parallelism是动态概念**：就是实际使用taskManager的并发能力（其实就是说：**taskManger有很多的task slot，但是我规定只用少数几个，那么就算taskManager有很大的并发能力,但是因为限制只用其中一少部分，那么它的实际并发能力依然比较少**），可以通过该参数进行配置**parallelism.default**。（举例说明：假设一共有3个TaskManager，每一个TaskManager中的分配3个TaskSlot，也就是每个TaskManager可以接收3个task，一共9个TaskSlot，如果我们设置parallelism.default=1，即运行程序默认的并行度为1，9个TaskSlot只用了1个，有8个空闲，因此，设置合适的并行度才能提高效率）



#### 2. 下面这张图就是很好的解释task Slot与parallelism的概念关系

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c2a95b44d5645be8fd29fd0811cdcad~tplv-k3u1fbpfcp-watermark.image)

### 2. 程序与数据流

> flink程序都是由3部分组成，source，transformation，sink
>
> 1. source：读取数据源
> 2. transformation：各种算子处理加工
> 3. sink：负责输出

### 3. 执行图-ExecutionGraph

> 1. streamGraph:根据用户stream api生成最初的图，表示程序的拓扑结构
> 2. jobGraph:将符合条件的节点chain在一起，减少节点间的流动带来的序列化、反序列化、以及传输带来的消耗
> 3. executionGraph:jobGraph的并行化的版本
> 4. 物理执行图： JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个TaskManager 上部署 Task 后形成的“图”，并不是一个具体的数据结构。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a51ef80a85394c19aa80a255793d8bd0~tplv-k3u1fbpfcp-watermark.image)

### 4. 并行度

> 一个特定算子的子任务（subtask）个数,就是该算子的并行度。
>
> 一个流程序，其中某个算子的拥有最大的并行度，可以任务就是该流程序的并行度。

stream算子之间的传输有2种模式

1. one-to-one(类似spark中的窄依赖)

   > stream(比如在source和map operator之间)维护着分区以及元素的顺序

2. Redistributing(类似于spark中的宽依赖)

   > stream(map()跟keyBy/window之间或者keyBy/window跟sink之间)的分区会发生改变。每一个算子的子任务依据所选择的transformation发送数据到不同的目标任务

### 5. 任务链(Operator Chains)

> 相同并行度的one to one操作，Flink这样相连的算子链接在一起形成一个task，原来的算子成为里面的一部分。将算子链接成task是非常有效的优化：它能减少线程之间的切换和基于缓存区的数据交换，在减少时延的同时提升吞吐量。链接的行为可以在编程API中进行指定

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bb4d8873158477eaad210e42ad9ae70~tplv-k3u1fbpfcp-watermark.image)

