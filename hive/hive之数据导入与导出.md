# 1. 数据导入

> 向表中导入数据

## 1. 向表中装载数据（load）

**1. 语法**

```sql
load data [local] inpath '/opt/module/datas/student.txt' [overwrite] into table student [partition (partcol1=val1,…)];
```

**2. 说明**

> （1）load data:表示加载数据
>
> （2）local:表示从本地加载数据到hive表；否则从HDFS加载数据到hive表
>
> （3）inpath:表示加载数据的路径
>
> （4）overwrite:表示覆盖表中已有数据，否则表示追加
>
> （5）into table:表示加载到哪张表
>
> （6）student:表示具体的表
>
> （7）partition:表示上传到指定分区

## 2. 通过查询语句向表中插入数据（insert）

1. **关键信息**

> 1. insert into：以**追加数据的方式插入**到表或分区，原有数据不会删除
>
> 2. insert overwrite：会**覆盖表或分区**中已存在的数据
>
>    
>
>    **注意：insert不支持插入部分字段**

2. 类似MySQL的方式直接插入数据(**hive中很少用，几乎不用这种**)

   ```sql
   --创建测试分区表
   create table if not exists student_par(
       id int,
       name string
   )
   partitioned by (month string)
   row format delimited fields terminated by '\t';
   
   -- 插入数据
   insert into table  student_par partition(month='201709') values(1,'wangwu'),(2,'zhaoliu');
   ```

   

3. 单表查询插入

   ```sql
   insert into table student_par partition (month='202103')
   select id,name from stu_buck;
   ```

   

4. 多表/多分区插入模式（根据多张表查询结果）

   ```mysql
   from stu_buck -- 数据来源表
   insert into table student_par partition (month='202104') -- 待插入的
   select id,name  -- 字段顺序很重要，不然会导致数据错误,查询的字段顺序是会依次插入到建表时字段顺序中的
   insert into table student_par partition (month='202105')
   select id,name;
   ```

   

## 3. 通过查询语句创建表的同时插入数据（as select ）

```sql
create table if not exists student3
as select id, name from student;
```



## 4. 创建表示通过location关键字指定加载数据路径

```sql
create external table if not exists student5(
              id int, name string
              )
              row format delimited fields terminated by '\t'
              location '/student'; --指定数据位置,文件必须放在/student目录下
```



## 5. Import数据到指定的Hive表中

```sql
-- 注意：先用export导出后，再将数据导入
hive (default)> import table student2 partition(month='201709') from
 '/user/hive/warehouse/export/student';
```



## 6. 总结关键信息

> 1. load data [local] inpath 'path' into table tableName [partition(pname='name')];
> 2. insert into table tableName select c1,c2,.. from sourceTable;
> 3. insert overrite table tableName  select c1,c2,.. from sourceTable;



# 2. 数据导出

## 1. insert 导出（重点）

1. 导出到本地(有local)

   ```sql
   -- 查询结果导出到Linux本地
   insert overwrite local directory '/opt/module/datas/export/student'
               select * from student;
               
   -- 带格式的导出到本地
   insert overwrite local directory '/opt/module/datas/export/student1'
              ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'            
              select * from student;
   ```

   

2. 导出到hdfs(没有local)

   ```sql
   insert overwrite directory '/user/atguigu/student2'
                ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
                select * from student;
   ```

   

## 2. hadoop命令导出

```shell
dfs -get /user/hive/warehouse/student/month=201709/000000_0
/opt/module/datas/export/student3.txt;
```



## 3. hive shell命令导出

```shell
# 基本语法：（hive -f/-e 执行语句或者脚本 > file）
bin/hive -e 'select * from default.student;' >
 /opt/module/datas/export/student4.txt;
```



## 4. export 导出到hdfs

```shell
# export和import主要用于两个Hadoop平台集群之间Hive表迁移。
export table default.student to
 '/user/hive/warehouse/export/student';
```



## 5. sqoop导出