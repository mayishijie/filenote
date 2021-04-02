# 1. mr跑的慢的原因

主要性能瓶颈有2大块

1. 计算机性能

   > cup,内存，磁盘，网络

2. I/O操作优化

   > 1. 数据倾斜
   > 2. map和reduce数设置不合理
   > 3. map运行时间过长，导致reduce等待时间过长
   > 4. 小文件过多
   > 5. 大量的不可分块的超大文件
   > 6. spill次数过多
   > 7. merge次数过多

# 2. mr优化6大方面

## 1. 数据输入

> 1. 合并小文件（在执行mr之前，写个程序合并小文件或者使用下面的第二种方式，它也会合并在读取数据的时候合并小文件）
> 2. 采用CombineTextInputForamt作为输入
>
> 总结关键词：**合并小文件**

## 2. map阶段 

> 1. 减少溢写（spill）次数：
>
>    mapreduce.task.io.sort.mb:内存缓冲区的默认大小是100M
>
>    mapreduce.map.sort.spill.percent: 默认值是0.8
>
>    **调整这两个参数，增大触发spill内存上限，减少磁盘I/O**.
>
> 2. 减少合并（merge）次数：
>
>    mapreduce.task.**io.sort.factor**: 默认值是10，默认spill文件数达到10个，将会进行merge
>
>    **增大改值，可以减少merge次数**
>
> 3. 在map之后，在不影响业务的前提下(中位数这些场景，肯定不能combine处理了)，**进行combine处理**，减少I/O
>
> 总结关键词：**减少I/O**

## 3. reduce阶段

> 1. 设置合理的map和reduce数
>
> 2. 设置map和reduce共存
>
>    mapreduce.job.reduce.slowstart.completedmaps：调整该参数，让map运行到一定成都后，reduce也开始运行
>
> 3. 规避使用reduce
>
>    reduce在连接数据集的时候会产生大量的网络I/O
>
> 4. 合理设置reduce端的buffer
>
>    mapreduce.reduce.input.buffer.percent:默认值0.0
>
>    当值大于0的时候，会保留指定比例的内存，读buffer中的数据直接拿给reduce使用。也需要根据业务场景来设置。
>
> 总结：

## 4. I/O传输

> 1. 数据压缩，减少网络传输的I/O
> 2. 使用sequenceFile二进制文件

## 5. 数据倾斜问题

> 0. 数据倾斜现象：
>
>    a. 数据频率倾斜：某一个区域的数据量远远大于其它区域
>
>    b. 数据大小倾斜：部分记录的大小远远大于平均值
>
> 1. 减少数据倾斜的方法：
>
>    a. 抽样和范围分区: 通过抽样得到结果集来预设分区
>
>    b. 自定义分区：对于某个比较了解的行业或者业务，可以根据这些做自定义分区，将某些数据发送到固定的一部分reduce实例中。
>
>    c.  combine:预聚合
>
>    d.  采用map join,尽量避免reduce join
>
>    

## 6. 常用的调优参数

### 参数调优：

（1）以下参数是在用户自己的MR应用程序中配置就可以生效（mapred-default.xml）

| 配置参数                                      | 参数说明                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| mapreduce.map.memory.mb                       | 一个MapTask可使用的资源上限（单位:MB），默认为1024。如果MapTask实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.reduce.memory.mb                    | 一个ReduceTask可使用的资源上限（单位:MB），默认为1024。如果ReduceTask实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.map.cpu.vcores                      | 每个MapTask可使用的最多cpu core数目，默认值: 1               |
| mapreduce.reduce.cpu.vcores                   | 每个ReduceTask可使用的最多cpu core数目，默认值: 1            |
| mapreduce.reduce.shuffle.parallelcopies       | 每个Reduce去Map中取数据的并行数。默认值是5                   |
| mapreduce.reduce.shuffle.merge.percent        | Buffer中的数据达到多少比例开始写入磁盘。默认值0.66           |
| mapreduce.reduce.shuffle.input.buffer.percent | Buffer大小占Reduce可用内存的比例。默认值0.7                  |
| mapreduce.reduce.input.buffer.percent         | 指定多少比例的内存用来存放Buffer中的数据，默认值是0.0        |

（2）应该在YARN启动之前就配置在服务器的配置文件中才能生效（yarn-default.xml）

| 配置参数                                 | 参数说明                                        |
| ---------------------------------------- | ----------------------------------------------- |
| yarn.scheduler.minimum-allocation-mb     | 给应用程序Container分配的最小内存，默认值：1024 |
| yarn.scheduler.maximum-allocation-mb     | 给应用程序Container分配的最大内存，默认值：8192 |
| yarn.scheduler.minimum-allocation-vcores | 每个Container申请的最小CPU核数，默认值：1       |
| yarn.scheduler.maximum-allocation-vcores | 每个Container申请的最大CPU核数，默认值：32      |
| yarn.nodemanager.resource.memory-mb      | 给Containers分配的最大物理内存，默认值：8192    |

（3）Shuffle性能优化的关键参数，应在YARN启动之前就配置好（mapred-default.xml）

| 配置参数                         | 参数说明                          |
| -------------------------------- | --------------------------------- |
| mapreduce.task.io.sort.mb        | Shuffle的环形缓冲区大小，默认100m |
| mapreduce.map.sort.spill.percent | 环形缓冲区溢出的阈值，默认80%     |

### 容错相关参数(MapReduce性能优化)

| 配置参数                     | 参数说明                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| mapreduce.map.maxattempts    | 每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.reduce.maxattempts | 每个Reduce Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.task.timeout       | Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个Task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该Task处于Block状态，可能是卡住了，也许永远会卡住，为了防止因为用户程序永远Block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是600000。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是“AttemptID:attempt_14267829456721_123456_m_000224_0 Timed out after 300 secsContainer killed by the ApplicationMaster.”。 |

# 3. hdfs小文件优化

## 1.小文件弊端

>  hdfs每个文件都会在NameNode上创建一个索引，小文件过多，会产生很多索引，**一方面占用过多NameNode内存空间，另外一方面，索引过大，会降低索引速度。**

## 2.小文件解决

> 1. 数据采集时，将小文件合并在上传到hdfs上
>
> 2. 在业务处理之前，在hdfs上利用maprdeuce程序对小文件进行合并
>
> 3. 在mr处理时，可以采用combineTextInputFormat来处理，提高效率
>
> 4. 开启JVM重用（一个map运行在一个JVM上，开启重用的话，该map在JVM运行完毕的话，JVM会继续运行其它map）
>
>    具体设置：mapreduce.job.jvm.numtasks:值在10-20之间

