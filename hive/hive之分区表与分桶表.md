# 1. 分区表

## 1. 分区表概念理解

> **分区表实际上就是对应一个HDFS文件系统上的独立的文件夹**，该文件夹下是该分区所有的数据文件。**Hive中的分区就是分目录**，把一个大的数据集根据业务需要分割成小的数据集。在查询时通过WHERE子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多

## 2. 分区表创建

```sql
create table if not exists dept_partition
(
    deptno int,
    dname  string,
    loc    string
)
partitioned by (month string) --和普通的表创建不同,需要加上partitioned by,month分区目录
row format delimited fields terminated by '\t';
```

## 3. 加载数据到分区表中

> 注意：**分区表加载数据时，必须指定分区**

```sql
load data local inpath '/home/mayi/mayi_data/dept.txt'
into table mayi.dept_partition partition (month='20210405');
load data local inpath '/home/mayi/mayi_data/dept.txt'
into table mayi.dept_partition partition (month='20210406');
```

查看分区表数据

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94b5b91e38a14fbc8ee9a77b3ddbafac~tplv-k3u1fbpfcp-watermark.image)

## 4. 分区数据查询

```sql
-- 单分区查询
select * from dept_partition where month = '20210405';
-- 多分区联合查询
select * from dept_partition where month = '20210405';
union
select * from dept_partition where month = '20210406';
```

## 5. 新增分区(add partition)

```sql
--同时新增多个分区，直接后面加分区数
alter table dept_partition add partition (month='20210301') partition (month='20210302');
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/839f2e94a59443dc8b0bc4a6d6c72876~tplv-k3u1fbpfcp-watermark.image)

## 6. 删除分区(drop partition)

```sql
-- 可以同时删除多个分区
alter table dept_partition drop partition (month='20210301'), partition (month='20210302');
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6701d4d2b9b84ee2b9d88e61fe780f08~tplv-k3u1fbpfcp-watermark.image)

## 7. 查看分区数(show partitions tableName)

```shell
hive (mayi)> show partitions dept_partition;
OK
partition
month=20210405
month=20210406
Time taken: 0.666 seconds, Fetched: 2 row(s)
```



## 8. 查看分区表结构(desc formatted tableName)

```shell
hive (mayi)> desc formatted dept_partition;
OK
col_name	data_type	comment
# col_name            	data_type           	comment

deptno              	int
dname               	string
loc                 	string

# Partition Information
# col_name            	data_type           	comment

month               	string

# Detailed Table Information
Database:           	mayi
Owner:              	mayi
CreateTime:         	Tue Apr 06 09:49:20 CST 2021
LastAccessTime:     	UNKNOWN
Protect Mode:       	None
Retention:          	0
Location:           	hdfs://mayi101:9000/other/warehouse/mayi.db/dept_partition
Table Type:         	MANAGED_TABLE
Table Parameters:
	transient_lastDdlTime	1617673760

# Storage Information
SerDe Library:      	org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe
InputFormat:        	org.apache.hadoop.mapred.TextInputFormat
OutputFormat:       	org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat
Compressed:         	No
Num Buckets:        	-1
Bucket Columns:     	[]
Sort Columns:       	[]
Storage Desc Params:
	field.delim         	\t
	serialization.format	\t
Time taken: 0.207 seconds, Fetched: 34 row(s)
```

# 2. 分区表注意事项

## 1. 创建二级分区

```sql
create table if not exists dept_partition2
(
    deptno int,
    dname  string,
    loc    string
)
partitioned by (month string,day string) --和普通的表创建不同,需要加上partitioned by,month分区目录,day二级分区目录
row format delimited fields terminated by '\t';
```

## 2. 加载数据到二级分区

### 1. 正常加载

```sql
load data local inpath '/home/mayi/mayi_data/dept.txt'
into table mayi.dept_partition2 partition (month='202103',day='01');

--查询
select * from dept_partition2 where month = '202103' and day= '01';
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/610786ee426044a8a5c48a013e1eca05~tplv-k3u1fbpfcp-watermark.image)

### 2. 直接上传数据到分区目录，然后让分区表与数据产生关联

> 有下面三种方式实现
>
> 
>
> **一般直接创建好分区表后，直接load数据到对应的分区中,也就是上面的正常加载数据**

#### 1. 上传数据后修复

```sql
-- 1. 上传数据
hive (default)> dfs -mkdir -p
 /user/hive/warehouse/dept_partition2/month=201709/day=12;
hive (default)> dfs -put /opt/module/datas/dept.txt  /user/hive/warehouse/dept_partition2/month=201709/day=12;
-- 2. 查询数据，查询不到刚刚上传的数据
hive (default)> select * from dept_partition2 where month='201709' and day='12';
-- 3. 执行修复命令后可以查询到
hive> msck repair table dept_partition2;
-- 4. 再次查询，可以查询到
hive (default)> select * from dept_partition2 where month='201709' and day='12';
```



#### 2. 上传数据后添加分区

```sql
-- 1. 上传数据
hive (default)> dfs -mkdir -p
 /user/hive/warehouse/dept_partition2/month=201709/day=11;
hive (default)> dfs -put /opt/module/datas/dept.txt  /user/hive/warehouse/dept_partition2/month=201709/day=11;
-- 2. 添加分区
	hive (default)> alter table dept_partition2 add partition(month='201709',
 day='11');
 -- 3. 查询分区
 hive (default)> select * from dept_partition2 where month='201709' and day='11';
```



#### 3. 创建文件夹后load数据到分区

> 其实就是正常模式下，自己主动新建一个目录，其实没啥必要，直接load数据就行

```sql
-- 1. 创建文件夹
hive (default)> dfs -mkdir -p
 /user/hive/warehouse/dept_partition2/month=201709/day=10;
 -- 2. 上传数据
 hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table
 dept_partition2 partition(month='201709',day='10');
 -- 3. 查询数据
 hive (default)> select * from dept_partition2 where month='201709' and day='10';
```

# 3. 动态分区调整

> 关系型数据库中，对分区表Insert数据时候，数据库自动会根据分区字段的值，将数据插入到相应的分区中，Hive中也提供了类似的机制，即动态分区(Dynamic Partition)，只不过，使用Hive的动态分区，需要进行相应的配置。

## 1. 开启动态分区参数设置

```markdown
1. 开启动态分区功能（默认就是true，开启）
hive.exec.dynamic.partition=true

2. 设置为非严格模式（动态分区的模式，默认strict，表示必须指定至少一个分区为静态分区，nonstrict模式表示允许所有的分区字段都可以使用动态分区。）
hive.exec.dynamic.partition.mode=nonstrict

3. 在所有执行MR的节点上，最大一共可以创建多少个动态分区。默认1000
hive.exec.max.dynamic.partitions=1000

4. `在每个执行MR的节点上，最大可以创建多少个动态分区`。该参数需要根据实际的数据来设定。比如：源数据中包含了一年的数据，即day字段有365个值，那么该参数就需要设置成大于365，如果使用默认值100，则会报错。
hive.exec.max.dynamic.partitions.pernode=100

5. 整个MR Job中，最大可以创建多少个HDFS文件。默认100000
hive.exec.max.created.files=100000

6. 当有空分区生成时，是否抛出异常。一般不需要设置。默认false
hive.error.on.empty.partition=false
```

## 2. 实例

> 需求：将dept表中的数据按照地区（loc字段），插入到目标表dept_partition的相应分区中。

1. 创建分区表

   ```sql
   create table if not exists dept_partition(
       id int, name string
   )
   partitioned by (location int)
   row format delimited fields terminated by '\t'
   ```

   

2. 设置动态分区

   ```sql
   --开启动态分区
   hive (mayi)> set hive.exec.dynamic.partition=true
   -- 设置为非严格模式
   hive (mayi)> set hive.exec.dynamic.partition.mode=nonstrict;
   -- 数据插入到分区中
   insert into table dept_partition partition (location)
   select deptno, dname, loc
   from dept;
   ```

   

3. 查看分区表情况

   ```sql
   hive (mayi)> show partitions dept_partition;
   OK
   partition
   location=1700
   location=1800
   location=1900
   Time taken: 0.163 seconds, Fetched: 3 row(s)
   hive (mayi)>
   ```

   ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcfe508c0c1f45239f026d859971c261~tplv-k3u1fbpfcp-watermark.image)

4. 注意

   > 目标分区表是如何匹配到分区字段的？
   >
   > 要点：因为dpartition表中只有两个字段，所以当我们查询了三个字段时（多了loc字段），所以系统默认以最后一个字段loc为分区名，因为分区表的
   > 分区字段默认也是该表中的字段，且依次排在表中字段的最后面。所以分区需要分区的字段只能放在后面，不能把顺序弄错。如果我们查询了四个字段的话，则会报
   > 错，因为该表加上分区字段也才三个。要注意系统是根据查询字段的位置推断分区名的，而不是字段名称。

# 4. 分桶表

> 分区提供一个隔离数据和优化查询的便利方式。不过，并非所有的数据集都可形成合理的分区。对于一张表或者分区，Hive 可以进一步组织成桶，也就是更为细粒度的数据范围划分。
>
> 分桶是将数据集分解成更容易管理的若干部分的另一个技术。
>
> **分区针对的是数据的存储路径**；
>
> **分桶针对的是数据文件**。

## 1. 创建分桶表，导入数据（不能使用load data）

```sql
--创建分桶表
create table stu_buck(
    id int,
    name string
)
clustered by (id) into 4 buckets --指定按照id值分到4个通里面去
row format delimited fields terminated by '\t';
```



注意: load data 不适合直接导入到分桶表

```sql
-- 我们知道,对于分桶表,是不能使用 load data 的方式进行插入数据的操作的,因为load data 导入数据不会有分桶结构.
-- 为了避免针对桶表使用load data 进行插入数据的操作,我们可以限制对桶表进行load操作, 		
set    hive.strict.checks.bucketing = true;

--	也可以在CM的hive配置项中修改此配置，当针对桶表执行load    data操作时会报错。 		
--	针对文本数据,想要导入到Hive分桶表中,我们可以先建立一个临时表,通过load data    将TXT文本导入到临时表中,

--创建临时表
create table temp_buck(id int, name string)
row format delimited fields terminated by '\t';
--导入数据
load data local inpath '/tools/test_buck.txt' into table temp_buck;
1
2
3
4
5
然后再在Hive中建立一个具有分桶结构的表,开启强制分桶,
使用insert  select 语句间接地把数据从临时表导入到分桶表中.
1
2
`--启用桶表
set hive.enforce.bucketing=true;
--限制对桶表进行load操作
set hive.strict.checks.bucketing = true;
--insert select
insert into table test_buck select id, name from temp_buck;
--分桶成功`

```

## 2. 创建分桶表时，数据通过子查询的方式导入

```sql
--（1） 先建一个普通的stu表

create table temp_buck(
    id int,
    name string
)
row format delimited fields terminated by '\t';

--（2） 向普通的stu表中导入数据

load data local inpath '/home/mayi/mayi_data/student.txt' into table temp_buck;

--（3） 清空stu_buck表中数据

truncate table stu_buck;

select * from stu_buck;

--（4） 导入数据到分桶表，通过子查询的方式

insert into table stu_buck

select id, name from temp_buck;

--（5） 发现还是只有一个分桶，如下图所示
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbf7aa5c22d0437a86834f92a2524abb~tplv-k3u1fbpfcp-watermark.image)

## 3. 特别注意

```
hive (mayi)> set hive.enforce.bucketing=true;--启动分桶
hive (mayi)> insert into stu_buck select id,name from temp_buck;

备注：
--限制对桶表进行load操作
set hive.strict.checks.bucketing = true;
--不开启reduce
hive (default)> set mapreduce.job.reduces=-1;
```



![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b5e606ab6ab48e49bd6cf97eced2298~tplv-k3u1fbpfcp-watermark.image)