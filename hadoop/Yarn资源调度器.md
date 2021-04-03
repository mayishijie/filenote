前言
>Yarn是一个**资源调度平台**，负责**为运算程序提供服务器运算资源**，相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于运行于操作系统之上的应用程序。
# 1. Yarn基本架构

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/973f327c254f4d1b9ec95567b3956ca3~tplv-k3u1fbpfcp-watermark.image)
## 1. RM(ResourceManager)-Yarn调度中心的总大脑
> 1. 处理客户端请求
> 2. 监控NodeManager
> 3. 启动或监控ApplicationMaster
> 4. 资源的分配与调度

## 2. NM(NodeManager)-Yarn分支上的大脑

> 1. 管理单个节点上的资源
> 2. 处理来自ResourceManager上的命令
> 3. 处理来自ApplicationMaster的命令

## 3. ApplicationMaster-任务的头

> 1. 负责数据的切分
> 2. 为应用程序申请资源并分配给内部任务
> 3. 任务的监控与容错

## 4. Container-身体

> 它是yarn上资源抽象，封装了CPU，内存，网络等资源

# 2. yarn工作机制

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ad86d8cccce4698a300917632bd7cf1~tplv-k3u1fbpfcp-watermark.image)

## 2.1 yarn工作机制详细过程

> ​	（1）MR程序提交到客户端所在的节点。
>
> ​	（2）YarnRunner向ResourceManager申请一个Application。
>
> ​	（3）RM将该应用程序的资源路径返回给YarnRunner。
>
> ​	（4）该程序将运行所需资源提交到HDFS上。
>
> ​	（5）程序资源提交完毕后，申请运行mrAppMaster。
>
> ​	（6）RM将用户的请求初始化成一个Task。
>
> ​	（7）其中一个NodeManager领取到Task任务。
>
> ​	（8）该NodeManager创建容器Container，并产生MRAppmaster。
>
> ​	（9）Container从HDFS上拷贝资源到本地。
>
> ​	（10）MRAppmaster向RM 申请运行MapTask资源。
>
> ​	（11）RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。
>
> ​	（12）MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。
>
> ​	（13）MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。
>
> ​	（14）ReduceTask向MapTask获取相应分区的数据。
>
> ​	（15）程序运行完毕后，MR会向RM申请注销自己。

## 2.2 mr作业提交全流程

### 流程图和上面的一致

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ad86d8cccce4698a300917632bd7cf1~tplv-k3u1fbpfcp-watermark.image)

### 作业提交流程详解

1. 作业提交

   > 第1步：Client调用job.waitForCompletion方法，向整个集群提交MapReduce作业。
   >
   > 第2步：Client向RM申请一个作业id。
   >
   > 第3步：RM给Client返回该job资源的提交路径和作业id。
   >
   > 第4步：Client提交jar包、切片信息和配置文件到指定的资源提交路径。
   >
   > 第5步：Client提交完资源后，向RM申请运行MrAppMaster。

2. 作业初始化

   > 第6步：当RM收到Client的请求后，将该job添加到容量调度器中。
   >
   > 第7步：某一个空闲的NM领取到该Job。
   >
   > 第8步：该NM创建Container，并产生MRAppmaster。
   >
   > 第9步：下载Client提交的资源到本地。

3. 任务分配

   > 第10步：MrAppMaster向RM申请运行多个MapTask任务资源。
   >
   > 第11步：RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

4. 任务运行

   > 第12步：MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。
   >
   > 第13步：MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。
   >
   > 第14步：ReduceTask向MapTask获取相应分区的数据。
   >
   > 第15步：程序运行完毕后，MR会向RM申请注销自己。

5. 进度和状态更新

   > YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户

6. 作业完成

   > 除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。

# 3. 资源调度器

目前Hadoop作业调度器主要有三种

> 1. FIFO：先进先出
> 2. Capacity Scheduler(Hadoop默认调度器)：容量调度器
> 3. Fair Scheduler：公平调度器

## 1. FIFO(先进先出)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1f7c810e9fe48abb71fb26d6d48f3e2~tplv-k3u1fbpfcp-watermark.image)

## 2. Capacity Scheduler（容量调度器）

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e471b55a8ae46aeaa135c411d7bf801~tplv-k3u1fbpfcp-watermark.image)

## 3. Fair Scheduler（公平调度器）

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fbbc7399b2e4f078dbd9e744e6352a6~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/649cea238acc45d3a1b092e936141e13~tplv-k3u1fbpfcp-watermark.image)

# 4. 容量调度器多队列配置

> 默认Yarn的配置下，容量调度器只有一条Default队列。在capacity-scheduler.xml中可以配置多条队列，并降低default队列资源占比

## 1. 修改配置

```xml
<property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,hive</value>
    <description>
      The queues at the this level (root is the root queue).
    </description>
</property>
<property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>40</value>
</property>
```

## 2. 配置新队列属性

```xml
<property>
    <name>yarn.scheduler.capacity.root.hive.capacity</name>
    <value>60</value>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
    <value>1</value>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
    <value>80</value>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.state</name>
    <value>RUNNING</value>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
    <value>*</value>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
    <value>*</value>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_application_max_priority</name>
    <value>*</value>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-application-lifetime</name>
    <value>-1</value>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.default-application-lifetime</name>
    <value>-1</value>
</property>
```

## 3. 配置完，重启Yarn即可

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5344a7db4fe4f179cb449e17ca0a30e~tplv-k3u1fbpfcp-watermark.image)

4. 指定任务提交到指定队列

   ```java
   configuration.set("mapred.job.queue.name", "hive");
   ```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e87b1fb012148b49a7714c0387f5549~tplv-k3u1fbpfcp-watermark.image)



