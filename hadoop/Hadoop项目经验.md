# 1. hdfs存储多目录
> 在hdfs存储空间紧张，我们需要对DataNode进行磁盘扩展，这里就涉及到hdfs存储多目录来处理这个问题。

具体步骤如下

1. 在DataNode节点增加磁盘，并进行挂载
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2237c22a8b34f3cbee8dfe3d74f2fc6~tplv-k3u1fbpfcp-watermark.image)
2. 在hdfs-site.xml文件中配置多目录，需要同时注意新磁盘的访问权限问题
```xml
<property>
    <name>dfs.datanode.data.dir</name>
<value>file:///${hadoop.tmp.dir}/dfs/data1,file:///hd2/dfs/data2,file:///hd3/dfs/data3,file:///hd4/dfs/data4</value>
</property>
```
3. 增加磁盘后，需要保证每个磁盘数据均衡

   1. 开启数据均衡命令

   > bin/start-balancer.sh –threshold 10

   解释：对于参数10，代表的是集群中各个节点的磁盘空间利用率相差不超过10%，可根据实际情况进行调整。

   2. 停止负载均衡:

	> bin/stop-balancer.sh

# 2. 配置LZO压缩

1. Hadoop-lzo编译

   > Hadoop支持LZO
   >
   > 0. 环境准备
   > maven（下载安装，配置环境变量，修改sitting.xml加阿里云镜像）
   > gcc-c++
   > zlib-devel
   > autoconf
   > automake
   > libtool
   > 通过yum安装即可，yum -y install gcc-c++ lzo-devel zlib-devel autoconf automake libtool
   >
   > 1. 下载、安装并编译LZO
   >
   > wget http://www.oberhumer.com/opensource/lzo/download/lzo-2.10.tar.gz
   >
   > tar -zxvf lzo-2.10.tar.gz
   >
   > cd lzo-2.10
   >
   > ./configure -prefix=/usr/local/hadoop/lzo/
   >
   > make
   >
   > make install
   >
   > 2. 编译hadoop-lzo源码
   >
   > 2.1 下载hadoop-lzo的源码，下载地址：https://github.com/twitter/hadoop-lzo/archive/master.zip
   > 2.2 解压之后，修改pom.xml
   >     <hadoop.current.version>2.7.2</hadoop.current.version>
   > 2.3 声明两个临时环境变量
   >      export C_INCLUDE_PATH=/usr/local/hadoop/lzo/include
   >      export LIBRARY_PATH=/usr/local/hadoop/lzo/lib 
   > 2.4 编译
   >     进入hadoop-lzo-master，执行maven编译命令
   >     mvn package -Dmaven.test.skip=true
   > 2.5 进入target，hadoop-lzo-0.4.21-SNAPSHOT.jar 即编译成功的hadoop-lzo组件

2. 