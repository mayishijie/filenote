# 1. shuffle源码

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8918d6217e5f4d958a376916b792fdf4~tplv-k3u1fbpfcp-watermark.image)
# 2. shuffle流程图

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9166ec98cfd64ca8bd4bba832e97261f~tplv-k3u1fbpfcp-watermark.image)
# 3. shuffle过程
> 1. MapTask收集我们的map()方法的k,v对，并将map结果和partition的结果缓存在内存缓冲区中。(pindex,k,v)三元组形式
> 2. 内存缓冲区达到阈值时(默认是100M，阈值默认是80%)，溢写线程锁住这80M缓冲区数据，并溢写到磁盘，每次溢写都会生成一个数据文件。**溢写之前都会先对KEY进行sort排序以及合并combiner(如果开启了combinr)**
> 3. 多个溢出的文件会被合并成一个大的溢出文件。
> 4. 在溢出过程以及合并过程，都要调用partitioner分区以及对key进行排序
> 5. ReduceTask根据自己的分区号，去各个MapTask机器上取相应的结果分区数据
> 6. ReduceTask会取到同一个分区的来自不同MapTask的结果文件，ReduceTask会将这些文件再进行合并（归并排序）
> 7. 合并成大文件后，Shuffle的过程也就结束了，后面进入ReduceTask的逻辑运算过程（从文件中取出一个一个的键值对Group，调用用户自定义的reduce()方法）
> 
> 注意：
> *（1）Shuffle中的缓冲区大小会影响到MapReduce程序的执行效率，原则上说，缓冲区越大，磁盘io的次数越少，执行速度就越快。
> *（2）缓冲区的大小可以通过参数调整，参数：io.sort.mb默认100M。