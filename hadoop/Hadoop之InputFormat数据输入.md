# 1. 切片与MapTask并行度决定机制
## 1. MapTask并行度决定机制
> * 数据块：Block是HDFS物理上把数据分成一块一块。
> * 数据切片：数据切片只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。
重要概念：
> 1. 一个job的map阶段的并行度由客户端在提交job时的切片数决定的。(`切片数=map数`)
> 2. 每一个split切片分配一个maptask实例并行处理。
> 3. 默认情况 切片大小=BlockSize
> 4. 切片时，不考虑整体集数据，而是针对每一个文件单独切片。

## 2. job提交源码以及切片源码关键信息
```java
waitForCompletion()

submit();

// 1建立连接
connect();	
    // 1）创建提交Job的代理
new Cluster(getConfiguration());
        // （1）判断是本地yarn还是远程
        initialize(jobTrackAddr, conf); 

// 2 提交job
submitter.submitJobInternal(Job.this, cluster)
    // 1）创建给集群提交数据的Stag路径
Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);

    // 2）获取jobid ，并创建Job路径
    JobID jobId = submitClient.getNewJobID();

    // 3）拷贝jar包到集群
copyAndConfigureFiles(job, submitJobDir);	
	rUploader.uploadFiles(job, jobSubmitDir);
    // 4）计算切片，生成切片规划文件
writeSplits(job, submitJobDir);
		maps = writeNewSplits(job, jobSubmitDir);
		input.getSplits(job);

// 5）向Stag路径写XML配置文件
writeConf(conf, submitJobFile);
	conf.writeXml(out);

// 6）提交Job,返回提交状态
status = submitClient.submitJob(jobId, submitJobDir.toString(), 
job.getCredentials());
```
# 2. FileInputFormat切片源码解析
1. 切片源码解析
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d575087529eb48b8be40dc16d0a615c1~tplv-k3u1fbpfcp-watermark.image)

2. FileInputFormat切片机制

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b5b9278dced14c368b162d643c4dc888~tplv-k3u1fbpfcp-watermark.image)
3. 切片参数设置

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcd40dc0f65c46acbf56f468d17f75c0~tplv-k3u1fbpfcp-watermark.image)

# 3. CombineTextInputFormat切片机制
1. 框架默认的切片机制：TextInPutFormat
> 框架默认的TextInputFormat切片机制是对任务按文件规划切片，`不管文件多小，都会是一个单独的切片`，都会交给一个MapTask，这样如果有大量小文件，就会产生大量的MapTask，处理效率极其低下
2. 应用场景
> CombineTextInputFormat切片机制，可以应用于小文件过多的场景，它可以将多个小文件从逻辑规划到一个切片上。这样多个小文件就可以交给一个maptask来处理。
3. 虚拟存储切片的最大值
> * CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m
> * 注意：虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值
4. 切片过程
生成切片过程包括：`虚拟存储过程`和`切片过程`二部分
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a602ee5a028a441e8b488b50263f6d4a~tplv-k3u1fbpfcp-watermark.image)

5. **实际操作使用关键点**

    1.  驱动类中指定切片类
    ```java
    // 如果不设置InputFormat，它默认用的是TextInputFormat.class
    job.setInputFormatClass(CombineTextInputFormat.class);

    //虚拟存储切片最大值设置4m
    CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);
    ```
# 4. inputFormat总结
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/623aa79cb6dd43dfb51cb2a133ed91bc~tplv-k3u1fbpfcp-watermark.image)

1. TextInputFormat:

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b904b113dd142c0abbbc82fe61c1b19~tplv-k3u1fbpfcp-watermark.image)

2. 其它
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4bc48b459654fc5acec57464946bbf7~tplv-k3u1fbpfcp-watermark.image)
