# 1. 安装部署

```shell
# 1. 下载解压安装包
tar -zxf apache-flume-1.7.0-bin.tar.gz -C /opt/module/
# 2. 配置JAVA_HOME到flume配置中
vi flume-env.sh

export JAVA_HOME=/opt/module/jdk1.8.0_144
```



# 2. 监控端口数据-官方的小例子

## 1. 需求

> 首先启动Flume任务，监控本机44444端口，服务端；
>
> 然后通过netcat工具向本机44444端口发送消息，客户端；
>
> 最后Flume将监听的数据实时显示在控制台。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/000fbe14d5c74fc2896bc50fec624c51~tplv-k3u1fbpfcp-watermark.image)

## 2. 在flume安装目录下创建一个flume Agen的配置文件

```shell
# 1. 创建一个job目录
mkdir job
# 2. 创建一个配置文件flume-netcat-logger.conf
touch flume-netcat-logger.conf
vim flume-netcat-logger.conf
# 3. 添加如下内容

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

## 3. 配置文件解析

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42ed1ec303f04215a9264ac3850b9a12~tplv-k3u1fbpfcp-watermark.image)

## 4. 开启flume监听端口

```shell
# 第一种写法
bin/flume-ng agent --conf conf/ --name a1 --conf-file job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console

# 第二种写法
bin/flume-ng agent -c conf/ -n a1 –f job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console
```

参数解析说明:

> --conf conf/  ：表示配置文件存储在conf/目录
>
> --name a1	：表示给agent起名为a1
>
> --conf-file job/flume-netcat.conf ：flume本次启动读取的配置文件是在job文件夹下的flume-telnet.conf文件。
>
> -Dflume.root.logger==INFO,console ：-D表示flume运行时动态修改flume.root.logger参数属性值，并将控制台日志打印级别设置为INFO级别。日志级别包括:log、info、warn、error。

# 3. 实时读取本地文件到hdfs上

> 需求：实时监控Hive日志，并上传到HDFS中

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/797063567273467692b9e599697dc94a~tplv-k3u1fbpfcp-watermark.image)

## 实现步骤

1. Flume要想将数据输出到HDFS，必须持有Hadoop相关jar

   > commons-configuration-1.6.jar、
   >
   > hadoop-auth-2.7.2.jar、
   >
   > hadoop-common-2.7.2.jar、
   >
   > hadoop-hdfs-2.7.2.jar、
   >
   > commons-io-2.4.jar、
   >
   > htrace-core-3.1.0-incubating.jar

2. 创建flume-file-hdfs.conf文件

   > **注：**要想读取Linux系统中的文件，就得按照Linux命令的规则执行命令。由于Hive日志在Linux系统中所以读取文件的类型选择：exec即execute执行的意思。表示执行Linux命令来读取文件

   ```shell
   # 1. 创建文件
   touch flume-file-hdfs.conf
   vim flume-file-hdfs.conf
    
    # 2. 添加如下内容
    # Name the components on this agent
   a2.sources = r2
   a2.sinks = k2
   a2.channels = c2
   
   # Describe/configure the source
   a2.sources.r2.type = exec
   a2.sources.r2.command = tail -F /opt/software/apache-hive-1.2.2-bin/logs/mayi/hive.log
   a2.sources.r2.shell = /bin/bash -c
   
   # Describe the sink
   a2.sinks.k2.type = hdfs
   a2.sinks.k2.hdfs.path = hdfs://mayi101:9000/flume/%Y%m%d/%H
   #上传文件的前缀
   a2.sinks.k2.hdfs.filePrefix = logs-
   #是否按照时间滚动文件夹
   a2.sinks.k2.hdfs.round = true
   #多少时间单位创建一个新的文件夹
   a2.sinks.k2.hdfs.roundValue = 1
   #重新定义时间单位
   a2.sinks.k2.hdfs.roundUnit = hour
   #是否使用本地时间戳
   a2.sinks.k2.hdfs.useLocalTimeStamp = true
   #积攒多少个Event才flush到HDFS一次
   a2.sinks.k2.hdfs.batchSize = 1000
   #设置文件类型，可支持压缩
   a2.sinks.k2.hdfs.fileType = DataStream
   #多久生成一个新的文件
   a2.sinks.k2.hdfs.rollInterval = 60
   #设置每个文件的滚动大小
   a2.sinks.k2.hdfs.rollSize = 134217700
   #文件的滚动与Event数量无关
   a2.sinks.k2.hdfs.rollCount = 0
   
   # Use a channel which buffers events in memory
   a2.channels.c2.type = memory
   a2.channels.c2.capacity = 1000
   a2.channels.c2.transactionCapacity = 100
   
   # Bind the source and sink to the channel
   a2.sources.r2.channels = c2
   a2.sinks.k2.channel = c2
   ```

   ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec227bd78d0e424c8abdef97da058d52~tplv-k3u1fbpfcp-watermark.image)

   > 注意：
   >
   > 1. 对于所有与时间相关的转义序列，Event Header中必须存在以 “timestamp”的key（除非hdfs.useLocalTimeStamp设置为true，此方法会使用TimestampInterceptor自动添加timestamp）。
   >
   > **a3.sinks.k3.hdfs.useLocalTimeStamp = true**

3. 启动flume

   ```shell
   [mayi@mayi101 apache-flume-1.7.0-bin]$ bin/flume-ng agent  -c conf/ -n a2 -f job/flume-file-hdfs.conf
   ```

4. hdfs上查看flume采集的数据

   ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9e442fd601f48eab137cd8358ff01dc~tplv-k3u1fbpfcp-watermark.image)

# 4. 实时读取目录文件到hdfs上

> 使用Flume监听整个目录的文件

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4cb08c6db17c445e84afbf4ee2acc374~tplv-k3u1fbpfcp-watermark.image)

配置文件

```shell
# 新增监控目录的配置文件
vim flume-dir-hdfs.conf
# 添加如下内容
a3.sources = r3
a3.sinks = k3
a3.channels = c3

# Describe/configure the source
a3.sources.r3.type = spooldir			#spooldir
a3.sources.r3.spoolDir = /opt/module/flume/upload		
a3.sources.r3.fileSuffix = .COMPLETED
a3.sources.r3.fileHeader = true
#忽略所有以.tmp结尾的文件，不上传
a3.sources.r3.ignorePattern = ([^ ]*\.tmp)

# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = hdfs://hadoop102:9000/flume/upload/%Y%m%d/%H
#上传文件的前缀
a3.sinks.k3.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a3.sinks.k3.hdfs.round = true
#多少时间单位创建一个新的文件夹
a3.sinks.k3.hdfs.roundValue = 1
#重新定义时间单位
a3.sinks.k3.hdfs.roundUnit = hour
#是否使用本地时间戳
a3.sinks.k3.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a3.sinks.k3.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a3.sinks.k3.hdfs.fileType = DataStream
#多久生成一个新的文件
a3.sinks.k3.hdfs.rollInterval = 60
#设置每个文件的滚动大小大概是128M
a3.sinks.k3.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a3.sinks.k3.hdfs.rollCount = 0

# Use a channel which buffers events in memory
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cca4142f462d4b1bbc99d5639270bf78~tplv-k3u1fbpfcp-watermark.image)

# 5. 单数据源多出口

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42506376455340b3b0cc3cc40760e5ca~tplv-k3u1fbpfcp-watermark.image)

## 需求

> 使用Flume-1监控文件变动，Flume-1将变动内容传递给Flume-2，Flume-2负责存储到HDFS。同时Flume-1将变动内容传递给Flume-3，Flume-3负责输出到Local FileSystem。

## 分析

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b0e6382f5734778bfe7fe2a5752be80~tplv-k3u1fbpfcp-watermark.image)

