# 1. MapTask工作机制
## 1. mapTask流程图
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fc6a8e636034db0ad8cd9741607c03e~tplv-k3u1fbpfcp-watermark.image)
## 2. 详解
1. read阶段
  
    > MapTask通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。
2. Map阶段
  
    > 该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value
3. Collect收集阶段
  
    > 在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。(partition,key,value)三元组的形式
4. Spill阶段
    > 即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。
    > 溢写阶段详情
    >
    > 1. 利用**快速排序算法对缓存区内的数据进行排序**，排序方式是，**先按照分区**编号Partition进行排序，**然后按照key**进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。
    
    > 2. 按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。
    
    > 3. 将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。
5. Combine阶段
  
    > 当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。

## 3. 说明：
> * 当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。
> * 在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并io.sort.factor（默认10）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。
> * 让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销

# 2. ReduceTask工作机制
## 1. reduceTask工作流程图

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98cc60a99edf4418ab78a1d3e7bdb946~tplv-k3u1fbpfcp-watermark.image)
## 2. 详解过程
1. Copy阶段
  
    > ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。
2. Merge阶段
  
    > 在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。
3. Sort阶段
  
    > 按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。
4. Reduce阶段
  
    > reduce()函数将计算结果写到HDFS上。

## 3. reduce并行度
MapTask的并发度是由切片数决定的，reduceTask的数量是可以手动指定的
> job.setNumReduceTasks(4);// 默认值是1，手动设置为4
## 4. 注意事项

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f232e7ef492e411c93cd9a5dd9c50dd0~tplv-k3u1fbpfcp-watermark.image)