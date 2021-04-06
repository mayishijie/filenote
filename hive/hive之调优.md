# 1. Fatch抓取

> set hive.fetch.task.conversion=more;

> Fetch抓取是指，**Hive中对某些情况的查询可以不必使用MapReduce计算**。例如：SELECT * FROM employees;在这种情况下，Hive可以简单地读取employee对应的存储目录下的文件，然后输出查询结果到控制台。
>
> 在hive-default.xml.template文件中**hive.fetch.task.conversion**默认是more，老版本hive默认是minimal，该属性修改为more以后，在全局查找、字段查找、limit查找等都不走mapreduce。

```xml
<property>
    <name>hive.fetch.task.conversion</name>
    <value>more</value>
    <description>
      Expects one of [none, minimal, more].
      Some select queries can be converted to single FETCH task minimizing latency.
      Currently the query should be single sourced not having any subquery and should not have any aggregations or distincts (which incurs RS), lateral views and joins.
      0. none : disable hive.fetch.task.conversion
      1. minimal : SELECT STAR, FILTER on partition columns, LIMIT only
      2. more  : SELECT, FILTER, LIMIT only (support TABLESAMPLE and virtual columns)
    </description>
  </property>
```

注意

```shell
set hive.fetch.task.conversion=none; # 所有查询都会执行mr
set hive.fetch.task.conversion=minimal;#
set hive.fetch.task.conversion=more;#全局，字段，limit查找都不走mr
```

# 2. 本地模式

> 大多数的Hadoop Job是需要Hadoop提供的完整的可扩展性来处理大数据集的。不过，有时Hive的输入数据量是非常小的。在这种情况下，为查询触发执行任务消耗的时间可能会比实际job的执行时间要多的多。**对于大多数这种情况，Hive可以通过本地模式在单台机器上处理所有的任务。对于小数据集，执行时间可以明显被缩短。**
>
> 
>
> 用户可以通过设置**hive.exec.mode.local.auto的值为true**，来让Hive在适当的时候自动启动这个优化，默认是false。

```shell
set hive.exec.mode.local.auto=true;  # 开启本地mr
# 设置local mr的最大输入数据量，当输入数据量小于这个值时采用local  mr的方式，默认为134217728，即128M
set hive.exec.mode.local.auto.inputbytes.max=50000000;
# 设置local mr的最大输入文件个数，当输入文件个数小于这个值时采用local mr的方式，默认为4
set hive.exec.mode.local.auto.input.files.max=10;
```

比对实际操作中开启本地模式时间对比

```shell
# 查看系统当前的属性值
hive (mayi)> set  hive.exec.mode.local.auto;
hive.exec.mode.local.auto=false # 默认是fasle，关闭了本地模式

# 关闭下的耗时
hive (mayi)> select * from emp cluster by deptno;
Time taken: 37.454 seconds, Fetched: 14 row(s)

# 开启本地模式
hive (mayi)> set hive.exec.mode.local.auto=true;


# 再次执行测试，开启下耗时
hive (mayi)> select * from emp cluster by deptno;
Time taken: 2.822 seconds, Fetched: 14 row(s)

```

**结论：**

> 开启本地模式，性能快了将近19倍

# 3. 表优化

## 1.小表大表join

## 2. 大表join大表

## 3. MapJoin(小表join大表)

## 4. group by

> 默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就倾斜了。
>
> 
>
>  并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果。

开启map端的聚合操作参数设置

```shell
#（1）是否在Map端进行聚合，默认为True
set hive.map.aggr = true
#（2）在Map端进行聚合操作的条目数目
set hive.groupby.mapaggr.checkinterval = 100000
#（3）有数据倾斜的时候进行负载均衡（默认是false）
set hive.groupby.skewindata = true
```



**set hive.groupby.skewindata = true**,这个参数很有用，在发生数据倾斜时

> **当选项设定为 true，生成的查询计划会有两个MR Job**。第一个MR Job中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是**相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的**；第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（**这个过程可以保证相同的Group By Key被分布到同一个Reduce中**），最后完成最终的聚合操作。

## 5. count(distinct)去重

> 数据量小的时候无所谓，数据量大的情况下，**由于COUNT DISTINCT的全聚合操作，即使设定了reduce task个数，set mapred.reduce.tasks=100；hive也只会启动一个reducer**。，这就造成一个Reduce处理的数据量太大，导致整个Job很难完成，**一般COUNT DISTINCT**使**用先GROUP BY再COUNT的方式替换：**

## 6. 笛卡尔积

> 尽量避免笛卡尔积，join的时候不加on条件，或者无效的on条件，Hive只能使用1个reducer来完成笛卡尔积。

## 7. 行列过滤

>  列处理：在SELECT中，**只拿需要的列**，**如果有，尽量使用分区过滤**，**少用SELECT** *。
>
> 行处理：在分区剪裁中，当使用外关联时，如果将副表的过滤条件写在Where后面，那么就会先全表关联，之后再过滤

实例

```sql
-- 1．测试先关联两张表，再用where条件过滤
hive (default)> select o.id from bigtable b
join ori o on o.id = b.id
where o.id <= 10;
Time taken: 34.406 seconds, Fetched: 100 row(s)

-- 2．通过子查询后，再关联表
hive (default)> select b.id from bigtable b
join (select id from ori where id <= 10 ) o on b.id = o.id;
Time taken: 30.058 seconds, Fetched: 100 row(s)
```



## 8. 动态分区调整

## 9. 分区，分桶

[参考hive之分区分桶](https://juejin.cn/post/6947880388656300045/)





# 4. 合理设置map和reduce数

## 1. 复杂文件增加map数

当input的文件都很大，任务逻辑复杂，map执行非常慢的时候，可以考虑增加Map数，来使得每个map处理的数据量减少，从而提高任务的执行效率。

增加map的方法为：根据computeSliteSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=128M公式，调整maxSize最大值。**让maxSize最大值低于blocksize就可以增加map的个数。**

```shell
# 设置最大切片值为100个字节
set mapreduce.input.fileinputformat.split.maxsize=100;
```



## 2. 小文件进行合并

```shell
#（1）在map执行前合并小文件，减少map数：CombineHiveInputFormat具有对小文件进行合并的功能（系统默认的格式）。HiveInputFormat没有对小文件合并功能。

set hive.input.format= org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;

#（2）在Map-Reduce的任务结束时合并小文件的设置：

#在map-only任务结束时合并小文件，默认true

SET hive.merge.mapfiles = true;

#在map-reduce任务结束时合并小文件，默认false

SET hive.merge.mapredfiles = true;

#合并文件的大小，默认256M

SET hive.merge.size.per.task = 268435456;

#当输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件merge

SET hive.merge.smallfiles.avgsize = 16777216;
```



## 3. 合理设置reduce

### 1. 调整reduce个数方法1

```shell
#（1）每个Reduce处理的数据量默认是256MB
hive.exec.reducers.bytes.per.reducer=256000000
#（2）每个任务最大的reduce数，默认为1009
hive.exec.reducers.max=1009
#（3）计算reducer数的公式
N=min(参数2，总输入数据量/参数1)
```



### 2. 调整reduce个数方法2

```shell
# 在hadoop的mapred-default.xml文件中修改
# 设置每个job的Reduce个数
set mapreduce.job.reduces = 15;
```

### 3. reduce不是越多越好

> 1）过多的启动和初始化reduce也会消耗时间和资源；
>
> 2）另外，有多少个reduce，就会有多少个输出文件，如果生成了很多个小文件，那么如果这些小文件作为下一个任务的输入，则也会出现小文件过多的问题；

在设置reduce个数的时候也需要考虑这两个原则：

> 1. 处理大数据量利用合适的reduce数；
>
> 2. 使单个reduce任务处理数据量大小要合适；

# 5. 并行执行

>  Hive会将一个查询转化成一个或者多个阶段。这样的阶段可以是MapReduce阶段、抽样阶段、合并阶段、limit阶段。或者Hive执行过程中可能需要的其他阶段。默认情况下，Hive一次只会执行一个阶段。不过，某个特定的job可能包含众多的阶段，而这些阶段可能并非完全互相依赖的，也就是说有些阶段是可以并行执行的，这样可能使得整个job的执行时间缩短。不过，如果有更多的阶段可以并行执行，那么job可能就越快完成。

​	通过设置参数hive.exec.parallel值为true，就可以开启并发执行。不过，在共享集群中，需要注意下，如果job中并行阶段增多，那么集群利用率就会增加。

```shell
set hive.exec.parallel=true;        //打开任务并行执行
set hive.exec.parallel.thread.number=16;  //同一个sql允许最大并行度，默认为8。
```

**当然，得是在系统资源比较空闲的时候才有优势，否则，没资源，并行也起不来。**



# 6. 严格模式

> Hive提供了一个严格模式，可以防止用户执行那些可能意想不到的不好的影响的查询。
>
> 通过设置属性hive.mapred.mode值为**默认是非严格模式nonstrict** 。开启严格模式需**要修改hive.mapred.mode值为strict**，开启严格模式可以禁止3种类型的查询。

```xml
<property>
    <name>hive.mapred.mode</name>
    <value>strict</value>
    <description>
      The mode in which the Hive operations are being performed. 
      In strict mode, some risky queries are not allowed to run. They include:
        Cartesian Product.
        No partition being picked up for a query.
        Comparing bigints and strings.
        Comparing bigints and doubles.
        Orderby without limit.
</description>
</property>
```

**禁止三种查询**

> 1) **对于分区表，除非where语句中含有分区字段过滤条件来限制范围，否则不允许执行**。换句话说，就是用户不允许扫描所有分区。进行这个限制的原因是，通常分区表都拥有非常大的数据集，而且数据增加迅速。没有进行分区限制的查询可能会消耗令人不可接受的巨大资源来处理这个表。
>
> 2) **对于使用了order by语句的查询，要求必须使用limit语句**。因为order by为了执行排序过程会将所有的结果数据分发到同一个Reducer中进行处理，强制要求用户增加这个LIMIT语句可以防止Reducer额外执行很长一段时间。
>
> 3) **限制笛卡尔积的查询。**对关系型数据库非常了解的用户可能期望在执行JOIN查询的时候不使用ON语句而是使用where语句，这样关系数据库的执行优化器就可以高效地将WHERE语句转化成那个ON语句。不幸的是，Hive并不会执行这种优化，因此，如果表足够大，那么这个查询就会出现不可控的情况。

# 7.JVM重用

> JVM重用是Hadoop调优参数的内容，其对Hive的性能具有非常大的影响，特别是**对于很难避免小文件的场景或task特别多的场景**，**这类场景大多数执行时间都很短。**
>
> 
>
> Hadoop的默认配置通常是使用派生JVM来执行map和Reduce任务的。这时JVM的启动过程可能会造成相当大的开销，尤其是执行的job包含有成百上千task任务的情况。**JVM重用可以使得JVM实例在同一个job中重新使用N次**。N的值可以在Hadoop的mapred-site.xml文件中进行配置。通常在10-20之间，具体多少需要根据具体业务场景测试得出。

```xml
<property>
  <name>mapreduce.job.jvm.numtasks</name>
  <value>10</value>
  <description>How many tasks to run per jvm. If set to -1, there is
  no limit. 
  </description>
</property>
```

缺点：

> 开启JVM重用将一直占用使用到的task插槽，以便进行重用，直到任务完成后才能释放。如果某个“不平衡的”job中有某几个reduce task执行的时间要比其他Reduce task消耗的时间多的多的话，那么保留的插槽就会一直空闲着却无法被其他的job使用，直到所有的task都结束了才会释放。

# 8.推测执行

# 9.压缩

# 10.执行计划