# 1. Hadoop和Spark中常用端口总结

1. Spark历史服务器端口号：18080	Hadoop的是19888
2. Spark master web端口：8080     Hadoop的NameNode web端口号50070（9870，3.x版本）
3. Spark master内部通信端口：7077  Hadoop的是9000端口（8020,3.x版本）
4. Spark查看当前spark-shell端口：4040
5. Hadoop YARN任务查看端口：8088

# 2. 分区规则

## 1. RDD数据从集合中创建

> 指定了分区数：则按照指定的分区数进行分区
>
> 未指定：按照机器核数创建分区

## 2. RDD数据从文件中创建

> math.min(取决于分配给应用的CPU的核数,2)

# 3. Transformation转换算子

## 1. value类型

### 1. map	一条数据

### 2. mapPartitions	一个分区的数据

> - 每次处理一个分区的数据，这个分区的数据处理完之后，原RDD中的数据才能释放，可能会导致OOM
>
> - 开发中，当内存空间比较大时，采用mapPartitions处理，提高效率

### 3. mapPartitionsWithIndex	带分区号

### 4. flatMap 打碎压平

### 5. glom 分区变数组

### 6. groupBy 分组，shuffle

### 7. filter 过滤，true留下，false的丢

### 8. sample 采样

### 9. distinct 去重 shuffle

> 用分布式的方式去重，比HashSet集合方式去重不容易OOM
>
> 内部实现原理：map(x => (x, null)).reduceByKey((x, y) => x, numPartitions).map(_._1)

### 10. coalesce 重新分区 (可配shuffle),减分区

> 默认不开启shuffle

### 11. repartition 重新分区 (执行shuffle)加减

> 本质：coalesce(numPartitions, shuffle = true)

### 12. sortBy 排序shuffle

> true-默认正序
>
> false-倒序

### 13. pipe 调用脚本

## 2. 双Value类型交互

### 1. union 并集

### 2. subtract 差集

### 3. intersection 交集

### 4. zip拉链，分区和元素个数必须相同

## 3.  key-value类型

### 1. partitionBy 按K重新分区

### 2. reduceByKey 按key聚合V shuffle 推荐

> 分区内先聚合,然后分区间

### 3. groupByKey 按key分组，shuffle，

> 根据key对RDD中的元素进行分组
>
> 比reduceByKey性能差点，reduceByKey有预聚合

### 4. aggregateByKey(初始值)(内,间) shuffle

> zeroValue 初始值，给分区内的每个元素分配一个初始值
>
> seqOp分区内：用初始值逐步迭代分区内的元素
>
> combOp分区间：合并每个分区的结果

### 5. foldByKey(初始值)(内间)

### 6. combineByKey((),(),())转换结构后分区内和分区间操作

>(
>
>(定义结构，类似于给每个元素使用了map坐转换),
>
>(对应结构下，分区的操作逻辑),
>
>(对应结构下，分区间的逻辑)
>
>)

### 7.  sortByKey 按key排序

> 如果key为自定义类型，要求必须混入Ordered特质
>
> ```scala
> class Student(var name:String,var age:Int) extends Ordered[Student] with Serializable {
>   //指定比较规则
>   override def compare(that: Student): Int = {
>     //先按照名称排序升序，如果名称相同的话，再按照年龄降序排序
>     var res: Int = this.name.compareTo(that.name)
>     if(res == 0){
>       res = that.age - this.age
>     }
>     res
>   }
>   override def toString = s"Student($name, $age)"
> }
> ```

### 8. mapValues 只对v操作，kv形式的数据

> map遍历，对每个value进行操作

### 9. join 相同key的多个value关联在一起 shuffle

> join算子相当于内连接，将两个RDD中的key相同的数据匹配，如果key匹配不上，那么数据不关联
>
> 结果是：（k,[所有分区匹配到的值]）

### 10. cogroup 类似全连接 shuffle

> 操作两个RDD中的KV元素，每个RDD中相同key中的元素分别聚合成一个集合
>
> 结果是：（k,[分区1所有v],[分区2所有v]）

# 4. Action行动算子

## 1. reduce 聚合

> 先聚合分区内所有数据，在聚合所有分区间数据

## 2. collect 数组形式返回数据集

> 所有的数据都会被拉到Driver端，谨慎使用

## 3. count 返回rdd元素个数

## 4. first 返回rdd第一个元素

## 5. take(n) 返回rdd前n个元素的数组

## 6. takeOrdered(n) 返回排序后的前n个元素

## 7. aggregate(initV)(内，间)

> initV:初始值（10）
>
> 分区内逻辑，每个元素都和初始值做分区内的逻辑
>
> 分区间逻辑，初始值只做一次和分区元素操作

## 8. fold(initV)(内间)

## 9. countByKey 统计每种key的个数

## 10. foreach 统计rdd中每个元素

## 11. save相关算子

### 1. saveAsTextFile(path)

### 2. saveAsSequenceFile(path)

### 3. saveAsObjectFile(path)

# 5.依赖关系

## 1. 血缘关系：toDebugString

## 2. 依赖关系：dependencies

## 3. 窄依赖

> 父rdd的一个分区会被子rdd的一个分区使用,分区1对1

## 4. 宽依赖

> 发生shuffle的基本都是宽依赖

# 6. spark的job调度

	.Spark的Job调度
		-集群(Standalone|Yarn)
			*一个Spark集群可以同时运行多个Spark应用
	-应用
		*我们所编写的完成某些功能的程序
		*一个应用可以并发的运行多个Job
	
	-Job
		*Job对应着我们应用中的行动算子，每次执行一个行动算子，都会提交一个Job
		*一个Job由多个Stage组成
	
	-Stage
		*一个宽依赖做一次阶段的划分
		*阶段的个数 =  宽依赖个数  + 1
		*一个Stage由多个Task组成
	
	-Task
		*每一个阶段最后一个RDD的分区数，就是当前阶段的Task个数

# 7. RDD持久化

	-cache
			底层调用的就是persist，默认存储在内存中
	-persist
		可以通过参数指定存储级别
	
	-checkpoint
		*可以当做缓存理解，存储在HDFS上，更稳定
		*为了避免容错执行时间过长
	
	-缓存不会切断血缘，但是检查点会切断血缘,都是在触发action算子时执行

# 8. 缓存和检查点区别

> 1. 缓存只是保存数据不切断血缘关系（缓存使用完成后，通过unpersist()释放），检查点不是
> 2. 缓存保存数据在内存或磁盘上，可靠性低，检查点在HDFS等高可用的文件系统中

# 9. 累加器与广播变量

## 1. 累加器

如果我们想实现所有分片处理时更新共享变量的功能，那么累加器可以实现我们想要的效果

> 分布式共享只写变量-------task之间不能读取数据

## 2. 广播变量

广播变量用来高效分发较大的对象

> 分布式共享只读变量
>
> 由之前发到每个task提升到excuter,节省内存空间