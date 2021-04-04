# 1. hdfs写流程

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43ae79eb4f414fe5a54d062755572684~tplv-k3u1fbpfcp-watermark.image)
# 2. hdfs读流程

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/451fdc740bff4a9d828a7b5a7cd2d2c3~tplv-k3u1fbpfcp-watermark.image)
# 3. 机架感知

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8c940878f1c41ebb1f8f130beddddc5~tplv-k3u1fbpfcp-watermark.image)
# 4. NN和2NN工作机制

（**Hadoop引入HA之后，就可以不用SecondaryNameNode了，因为主从备份的HADOOP-HA可以取代它**）

## 2NN的引入以及NN架构的思考
1. NameNode元数据存储位置：存储在内存
2. 为避免数据丢失，提高可靠性，还需要存放在磁盘中，**因此产生在磁盘中备份元数据的FsImage。**
3. 当在内存中的元数据更新时，如果同时更新FsImage，就会导致效率过低，但如果不更新，就会发生一致性问题，一旦NameNode节点断电，就会产生数据丢失。因此，引入**Edits文件**(只进行追加操作，效率很高)。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到Edits中。这样，一旦NameNode节点断电，可以通过FsImage和Edits的合并，合成元数据。
4. 如果长时间添加数据到Edits中，会导致该文件数据过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。因此，需要定期进行FsImage和Edits的合并，如果这个操作由NameNode节点完成，又会效率过低。因此，引入一个新的节点**SecondaryNamenode**，专门用于FsImage和Edits的合并
## NameNode工作机制

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f87bbe4b4104b27ab7993a7f2985efd~tplv-k3u1fbpfcp-watermark.image)

1. 第一阶段：NameNode启动

   > （1）第一次启动NameNode格式化后，创建Fsimage和Edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
   >
   > （2）客户端对元数据进行增删改的请求。
   >
   > （3）NameNode记录操作日志，更新滚动日志。
   >
   > （4）NameNode在内存中对元数据进行增删改。

2. 第二阶段：Secondary NameNode工作

   > （1）Secondary NameNode询问NameNode是否需要CheckPoint。直接带回NameNode是否检查结果。
   >
   > （2）Secondary NameNode请求执行CheckPoint。
   >
   > （3）NameNode滚动正在写的Edits日志。
   >
   > （4）将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode。
   >
   > （5）Secondary NameNode加载编辑日志和镜像文件到内存，并合并。
   >
   > （6）生成新的镜像文件fsimage.chkpoint。
   >
   > （7）拷贝fsimage.chkpoint到NameNode。
   >
   > （8）NameNode将fsimage.chkpoint重新命名成fsimage。

# 5. Fsimage和Edits解析

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f2f8371a2434b9b89deb3b2c486ee20~tplv-k3u1fbpfcp-watermark.image)

# 6.  CheckPoint时间设置

1. 通常情况SecondaryNameNode每隔一小时执行一次

   ```xml
   <!-- hdfs-default.xml -->
   <property>
     <name>dfs.namenode.checkpoint.period</name>
     <value>3600</value>
   </property>
   ```

   

2. 一分钟检查一次操作次数，3当操作次数达到1百万时，SecondaryNameNode执行一次。

   ```xml
   <property>
     <name>dfs.namenode.checkpoint.txns</name>
     <value>1000000</value>
   <description>操作动作次数</description>
   </property>
   
   <property>
     <name>dfs.namenode.checkpoint.check.period</name>
     <value>60</value>
   <description> 1分钟检查一次操作次数</description>
   </property >
   ```

   

# 7. NameNode故障处理

NameNode故障后，可采用如下2种方式恢复

1. 将SecondaryNameNode中数据拷贝到NameNode存储数据的目录；

   （1）kill -9 NameNode进程

   （2）删除NameNode存储的数据（/opt/module/hadoop-3.1.3/data/tmp/dfs/name）

   ```shell
   [atguigu@hadoop102 hadoop-3.1.3]$ rm -rf /opt/module/hadoop-3.1.3/data/tmp/dfs/name/*
   ```

   （3）拷贝SecondaryNameNode中数据到原NameNode存储数据目录

   ```shell
   [atguigu@hadoop102 dfs]$ scp -r atguigu@hadoop104:/opt/module/hadoop-3.1.3/data/tmp/dfs/namesecondary/* ./name/
   ```

   （4）重新启动NameNode

   ```shell
   [atguigu@hadoop102 hadoop-3.1.3]$ hdfs --daemon start namenode
   ```

2. 使用-importCheckpoint选项启动NameNode守护进程，从而将SecondaryNameNode中数据拷贝到NameNode目录中。

（1）修改hdfs-site.xml中的

```xml
<property>
  <name>dfs.namenode.checkpoint.period</name>
  <value>120</value>
</property>

<property>
  <name>dfs.namenode.name.dir</name>
  <value>/opt/module/hadoop-3.1.3/data/tmp/dfs/name</value>
</property>
```

（2）kill -9 NameNode进程

（3）删除NameNode存储的数据（/opt/module/hadoop-3.1.3/data/tmp/dfs/name）

```shell
[atguigu@hadoop102 hadoop-3.1.3]$ rm -rf /opt/module/hadoop-3.1.3/data/tmp/dfs/name/*
```

（4）如果SecondaryNameNode不和NameNode在一个主机节点上，需要将SecondaryNameNode存储数据的目录拷贝到NameNode存储数据的平级目录，并删除in_use.lock文件

```shell
[atguigu@hadoop102 dfs]$ scp -r atguigu@hadoop104:/opt/module/hadoop-3.1.3/data/tmp/dfs/namesecondary ./
[atguigu@hadoop102 namesecondary]$ rm -rf in_use.lock
[atguigu@hadoop102 dfs]$ pwd
/opt/module/hadoop-3.1.3/data/tmp/dfs
[atguigu@hadoop102 dfs]$ ls
data  name  namesecondary
```

（5）导入检查点数据（等待一会ctrl+c结束掉）

```shell
[atguigu@hadoop102 hadoop-3.1.3]$ bin/hdfs namenode -importCheckpoint
```

（6）启动NameNode

```shell
[atguigu@hadoop102 hadoop-3.1.3]$ hdfs --daemon start namenode
```

# 8. 集群安全模式
> 集群处于安全模式，不能执行重要操作（写操作）。集群启动完成后，自动退出安全模式。
>
> （1）bin/hdfs dfsadmin -safemode get		（功能描述：查看安全模式状态）
> （2）bin/hdfs dfsadmin -safemode enter  	（功能描述：进入安全模式状态）
> （3）bin/hdfs dfsadmin -safemode leave	（功能描述：离开安全模式状态）
> （4）bin/hdfs dfsadmin -safemode wait	（功能描述：等待安全模式状态）


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a69d1fd62fd4455bcaa7a3eb23e2616~tplv-k3u1fbpfcp-watermark.image)

# 9. DataNode工作机制

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b85c028bae434e3a90ce1397d6249ee8~tplv-k3u1fbpfcp-watermark.image)

## 1. 数据完整性

> 思考：如果电脑磁盘里面存储的数据是控制高铁信号灯的红灯信号（1）和绿灯信号（0），但是存储该数据的磁盘坏了，一直显示是绿灯，是否很危险？同理DataNode节点上的数据损坏了，却没有发现，是否也很危险，那么如何解决呢？

数据完整性保证的方法

> （1）当DataNode读取Block的时候，它会计算CheckSum。
>
> （2）如果计算后的CheckSum，与Block创建时值不一样，说明Block已经损坏。
>
> （3）Client读取其他DataNode上的Block。
>
> （4）DataNode在其文件创建后周期验证CheckSum。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8748ecf7635341c8bd9a0aeda79e5906~tplv-k3u1fbpfcp-watermark.image)

## 2. DataNode掉线时限设定

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23b1c2fbfb4543f487006af52d1418d4~tplv-k3u1fbpfcp-watermark.image)

## 3. 服役新数据节点

直接启动DataNode即可关联到集群

```shell
# 启动节点
hdfs --daemon start datanode
sbin/yarn-daemon.sh start nodemanager
```

如果数据不平衡，需要执行平衡脚本

```shell
./start-balancer.sh
```

## 4. 退役旧数据节点

### 1. 添加白名单

配置步骤

1. 在NameNode的/opt/module/hadoop-3.1.3/etc/hadoop目录下创建dfs.hosts文件

```shell
touch dfs.hosts
```

​	添加主机名（退役主机不要加进来）

```shell
mayi101
mayi102
```

2. hdfs-site.xml添加dfs.hosts属性

   ```xml
   <property>
   <name>dfs.hosts</name>
   <value>/opt/module/hadoop-3.1.3/etc/hadoop/dfs.hosts</value>
   </property>
   ```

3. 配置文件分发

   ```shell
   xsync hdfs-site.xml
   ```

   

4. 刷新nameNode

   ```shell
    hdfs dfsadmin -refreshNodes
   ```

5. 刷新ResourceManager

   ```shell
   yarn rmadmin -refreshNodes
   ```

   

6. web上可以查看



### 2. 黑名单退役

在黑名单上面的主机都会被强制退出。

类似于操作白名单

1. 在Hadoop目录下创建touch dfs.hosts.exclude 文件

2. 添加下架主机名mayi103

3. 配置hdfs-site.xml

   ```xml
   <property>
   <name>dfs.hosts.exclude</name>
         <value>/opt/module/hadoop-3.1.3/etc/hadoop/dfs.hosts.exclude</value>
   </property>
   ```

   

4. 刷新namenode

5. 刷新resourceManager

6. 等待该节点状态为decommissioned(web上可以查看)，停止进程

   ```shell
   hdfs --daemon stop datanode
   sbin/yarn-daemon.sh stop nodemanager
   ```

   

7. 

