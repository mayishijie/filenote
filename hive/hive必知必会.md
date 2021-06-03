# 1. Hive架构，可用引擎

MapReduce

Tez

Spark

# 2. Hive和数据库区别？

## 1. 相似性

> 类似的查询语言，其它没有任何相似性

## 2. 区别

### 1. 存储位置

>  hive数据存储在HDFS分布式文件系统中
>
> MySQL一般存储在块设备中

### 2. 数据更新

> Hive不推荐对数据改写
>
> MySQL经常数据改写

### 3. 执行延迟

> 整体来说，Hive执行延迟会高（基于MR特性），不过也是有前提的，如果是超大规模的数据，超出了数据库的处理能力，那么Hive的并行处理能力就会凸显出来了

### 4. 数据规模

> Hive可以支持的数据规模很大，tb，pb级别

# 3. 内部表和外部表

## 1. 内部表（管理表）

> 实际开发中，只有一些临时表会定义成管理表，删除的话元数据和表数据都会删除

## 2. 外部表

> 实际开发中，大部分都是定义成外部表，删除了表，仅仅是删除元数据，表数据还是存在的（**数据永远是非常重要的**）

# 4. 4个By

## 1. order  by

> 全局排序，只有一个Reduce，需要谨慎使用，容易发生数据倾斜

## 2. distrbute by

> 类似MR中的partition,进行分区，一般结合sort by使用，实际开发中使用该方式也是最多的

## 3. sort by

> 分区内有序，一般和distrbute by一起使用

## 4. cluster by

> 当distrbute by字段和sort by字段相同时，可以等价于distrbute by 和sort by使用，只是它只能升序排序，不能指定排序规则。

# 5. 窗口函数

## 1. rank()

> 排序： 1 1 3 4
>
> 相同会重复，总数不变

## 2. dense_rank():

> 排序： 1 1 2 3
>
> 相同会重复，会有缺

## 3. row_number():

> 排序： 1 2 3 4
>
> 排序正常，相同字段则会按照字段字典顺序排

## 4. 窗口 over

### over()

> 不带参数，默认是全窗口

```sql
select name,orderdate,cost, 
sum(cost) over() as sample1,--所有行相加 
sum(cost) over(partition by name) as sample2,--按name分组，组内数据相加 
sum(cost) over(partition by name order by orderdate) as sample3,--按name分组，组内数据累加 
sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING and current row ) as sample4 ,--和sample3一样,由起点到当前行的聚合 
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING and current row) as sample5, --当前行和前面一行做聚合 
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING AND 1 FOLLOWING ) as sample6,--当前行和前边一行及后面一行 
sum(cost) over(partition by name order by orderdate rows between current row and UNBOUNDED FOLLOWING ) as sample7 --当前行及后面所有行 
from business;
```





# 6. 自定义UDF,UDTF

## 1. 自定义UDF,继承UDF,重写evaluate

解析公共字段

## 2. 自定义UDTF,继承GenericUDTF

> initialize(自定义输出的列名和类型)
>
> process（将结果返回forward(result)）
>
> close

解析事件字段

## 3. 可以自己埋点，打印log,出错时可以方便调试

# 7. Hive优化

## 1. Fatch抓取，设置成more,这样全局，字段，limit等查找都不会走mr

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

## 2. 小表和大表join，小表放左边，先进内存，不过现在新版本的默认就是这样优化了，所以现在放左边和右边没啥区别

## 3. 大表Join大表

空key很容易到导致join到一个reduce，导致数据倾斜

### 1. 空Key过滤，先select from where id is not null

### 2. 空key转换，设置一个随机值

### 3. 开启MapJoin,让小表加载到内存，在map端join，避免reduce处理，发生数据倾斜

## 4. group by

### 1. 开启map端聚合

> set hive.map.aggr = true 	默认就是true

### 2. 设置 set hive.groupby.skewindata = true

> set hive.groupby.skewindata = true  避免数据倾斜，开启2段MR

## 5. 行列过滤

### 1. 列处理

> select 只拿需要的列，如果有分区，尽量使用分区过滤，少用select *

### 2. 行处理

> 在分区裁剪，使用外关联时，最好先对关联表的条件先过滤，然后用子查询进行关联

## 6. 合理设置Map和Reduce,针对MR引擎

## 7. 严格模式

hive.strict.checks.cartesian.product:true	笛卡尔积

hive.strict.checks.orderby.no.limit:true 	order by时，用limit，不用大量数据全表查

hive.strict.checks.no.partition.filter:true	要用分区过滤

## 8. JVM重用

小文件场景或者task特别多的场景

JVM重用可以使得JVM实例在同一个job中重新使用N次

缺点就是：一直占据资源

## 9. 压缩

在实际的项目开发当中，hive表的**数据存储**格式一般选择：**orc或parquet**。压缩方式一般选择**snappy，lzo**。

存储文件的压缩比：

ORC >  Parquet >  textFile

文件存储格式：

1. 行式存储

   > TEXTFILE和SEQUENCEFILE

   行查询块（数据靠近，基于存储的特点），数据压缩差点

2. 列式存储

>  orc和parquet

查询慢点，但是有很好的的压缩比

# 8. hive数据倾斜

# 9. 动态分区和静态分区

## 1. 区别

> a. 静态分区与动态分区的主要区别在于静态分区是手动指定，而动态分区是通过数据来进行判断。
>
> b. 详细来说，静态分区的列实在编译时期，通过用户传递来决定的；动态分区只有在 SQL 执行时才能决定。
>
> c. **动态分区是基于查询参数的位置去推断分区的名称，从而建立分区**

## 2. 动态分区

开启动态分区需要设置：非严格模式，开启动态分区的功能，默认就是true开启的

```sql
1. 开启动态分区功能（默认就是true，开启）
hive.exec.dynamic.partition=true

2. 设置为非严格模式（动态分区的模式，默认strict，表示必须指定至少一个分区为静态分区，nonstrict模式表示允许所有的分区字段都可以使用动态分区。）
hive.exec.dynamic.partition.mode=nonstrict
`在每个执行MR的节点上，最大可以创建多少个动态分区`。该参数需要根据实际的数据来设定。比如：源数据中包含了一年的数据，即day字段有365个值，那么该参数就需要设置成大于365，如果使用默认值100，则会报错。
hive.exec.max.dynamic.partitions.pernode=100
```

静态分区：是自己定义数据 多用于增量表（不断增加表的内容）比如新闻表，每天都会变化增加需要事先知道有多少分区，每个分区都要手动插入。

动态分区：事先先用group 或者distinct 看一下字段值的种类
多用于每年 每月 每日 等 多用于全量导入 数据量不能太大

# 10. hive中字段分隔符

> hive 默认的字段分隔符为ascii码的控制符\001（^A）,建表的时候用fields terminated by '\001'
>
> 遇到过字段里边有\t的情况，自定义InputFormat，替换为其他分隔符再做后续处理

# 11. 行转列：explode

> EXPLODE(col)：将hive一列中复杂的array或者map结构拆分成多行。
>
> LATERAL VIEW
>
> 用法：LATERAL VIEW udtf(expression) tableAlias AS columnAlias
>
> 解释：用于和split, explode等UDTF一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合。

```sql
--数据准备
create table if not exists movie(
    movie string,
    category string
)
row format delimited fields terminated by '\t';
load data local inpath '/home/mayi/mayi_data/movie.txt' into table movie;
--数据格式
movie.movie	movie.category
《疑犯追踪》	悬疑,动作,科幻,剧情
《Lie to me》	悬疑,警匪,动作,心理,剧情
《战狼2》	战争,动作,灾难

--需求：将电影分类数组展开
《疑犯追踪》      悬疑
《疑犯追踪》      动作
《疑犯追踪》      科幻
《疑犯追踪》      剧情
《Lie to me》   悬疑
《Lie to me》   警匪
《Lie to me》   动作
《Lie to me》   心理
《Lie to me》   剧情
《战狼2》        战争
《战狼2》        动作
《战狼2》        灾难
--实现
select
    m.movie,
    tbl.cate
from
    movie m
lateral view	-- 构造成一个多行的虚拟表
    explode(split(category, ",")) tbl as cate;

```



# 12 列转行: group by+concat,concat_ws,collection_set等聚合

> 所谓行转列，关键几个函数组合使用：**group by,concat,concat_ws,collect_set,collect_list**
>
> 1. group by可以将多行数据按照指定字段分组
> 2. concat,concat_ws将行中的某些字段进行汇总拼接成字符串
> 3. collect_set,collect_list将行中的某些字段汇总到一个集合中

```sql
-- 数据准备
create table if not exists person_info
(
    name          string,
    constellation string,
    blood_type    string
)
    row format delimited fields terminated by ',';
load data local inpath '/home/mayi/mayi_data/person_info.txt'
    into table person_info;
    
--数据格式
大海,射手座,A
宋宋,白羊座,B
猪八戒,白羊座,A
凤姐,射手座,A

--需求：把星座和血型一样的人归类到一起
--预期结果是：
射手座,A            大海|凤姐
白羊座,A            孙悟空|猪八戒
白羊座,B            宋宋|苍老师

-- 1. 星座，血型
hive (mayi)> select concat_ws(',',constellation,blood_type) cb,name from person_info;
OK
cb	name
射手座,A	大海
白羊座,B	宋宋
白羊座,A	猪八戒
射手座,A	凤姐
-- 2. 整合
select t.baseName, concat_ws('|', collect_set(t.name)) as name
from (
         select name, concat(constellation, ',', blood_type) baseName from person_info
     ) t
group by baseName

```

# 13. case when 和 if else  

```sql
-- 求出不同部门男女各多少人
select dept_id,
       sum(case sex when '男' then 1 else 0 end) man_sum,
       sum(case sex when '女' then 1 else 0 end) wo_sum
from emp_sex
group by dept_id;
--结果
deptid,man,woman
A,2,1
B,1,2

-- 文件准备,建表emp_sex
hive (mayi)> select * from emp_sex;
OK
emp_sex.name	emp_sex.dept_id	emp_sex.sex
悟空	A	男
大海	A	男
宋宋	B	男
凤姐	A	女
婷姐	B	女
婷婷	B	女


```

# 14. 日期函数

# 15. 分区和分桶

## 1. 分区-分目录，便于快速查询

**分区针对的是数据的存储路径**；

分区提供一个隔离数据和优化查询的便利方式

## 2. 分桶-更细粒度的划分

**分桶针对的是数据文件**。

### 分桶注意

1. 强制启动分桶

```
set hive.enforce.bucketing=true;--启动分桶
```

2. 限制load,只能insert

```
set hive.strict.checks.bucketing = true;
```

3. 不开启reduce，分桶时不需要开启reduce

```
hive (default)> set mapreduce.job.reduces=-1;
```

## 3. 可以对分桶进行抽样查询

```
select * from stu_buck tablesample(bucket 1 out of 4 on id);
```

