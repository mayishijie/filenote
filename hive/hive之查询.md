 

# 0. 数据准备

```sql
-- 创建部门表
create table if not exists dept
(
    deptno int,
    dname  string,
    loc    int
)
    row format delimited fields terminated by '\t';

-- 创建员工表
create table if not exists emp
(
    empno    int,
    ename    string,
    job      string,
    mgr      int,
    hiredate string,
    sal      double,
    comm     double,
    deptno   int
)
    row format delimited fields terminated by '\t';
-- 创建区域表
create table if not exists location(
	loc int,
  loc_name string
)
row format delimited fields terminated by '\t';

-- 查看创建的表
show tables;
-- 数据导入
load data local inpath '/home/mayi/mayi_data/emp.txt'
    into table emp;
load data local inpath '/home/mayi/mayi_data/dept.txt'
into table dept;
load data local inpath '/home/mayi/mayi_data/location.txt'
into table location;
```

# 1. 全表与特定列查询

```sql
-- 全表
hive (mayi)> select * from dept limit 3;
OK
dept.deptno	dept.dname	dept.loc
10	ACCOUNTING	1700
20	RESEARCH	1800
30	SALES	1900
Time taken: 0.157 seconds, Fetched: 3 row(s)
-- 特定列
hive (mayi)> select deptno,loc from dept;
OK
deptno	loc
10	1700
20	1800
30	1900
40	1700
Time taken: 0.18 seconds, Fetched: 4 row(s)
```

# 2. 常用函数

## 1. count:总数

```sql
-- 总行数
select count(*) cnt from emp;
```

## 2. max:最大值

```sql
--雇员最高工资
select max(sal) max_sal from emp;
```

## 3. min：最小值

```sql
-- 最低工资
select min(sal) min_sal from emp;
```

## 4. sum:总和

```sql
-- 工资总数
select sum(sal) sum_sal from emp; 
```

## 5. avg：平均值

```sql
-- 平均工资
select sum(sal) sum_sal from emp; 
```

# 3. limit ：限制返回行数

```sql
select * from emp limit 5;
```

# 4. where :条件过滤

```sql
select sal as tsal from emp where tsal >1000;//错误的方式，不能字段别名
```

> 注意点：
>
> 1. where子句中不能使用字段别名。

# 5. 比较运算符，和MySQL基本一致

| 操作符                  | 支持的数据类型 | 描述                                                         |
| ----------------------- | -------------- | ------------------------------------------------------------ |
| A=B                     | 基本数据类型   | 如果A等于B则返回TRUE，反之返回FALSE                          |
| A<=>B                   | 基本数据类型   | 如果A和B都为NULL，则返回TRUE，如果一边为NULL，返回False      |
| A<>B, A!=B              | 基本数据类型   | A或者B为NULL则返回NULL；如果A不等于B，则返回TRUE，反之返回FALSE |
| A<B                     | 基本数据类型   | A或者B为NULL，则返回NULL；如果A小于B，则返回TRUE，反之返回FALSE |
| A<=B                    | 基本数据类型   | A或者B为NULL，则返回NULL；如果A小于等于B，则返回TRUE，反之返回FALSE |
| A>B                     | 基本数据类型   | A或者B为NULL，则返回NULL；如果A大于B，则返回TRUE，反之返回FALSE |
| A>=B                    | 基本数据类型   | A或者B为NULL，则返回NULL；如果A大于等于B，则返回TRUE，反之返回FALSE |
| A [NOT] BETWEEN B AND C | 基本数据类型   | 如果A，B或者C任一为NULL，则结果为NULL。如果A的值大于等于B而且小于或等于C，则结果为TRUE，反之为FALSE。如果使用NOT关键字则可达到相反的效果。 |
| A IS NULL               | 所有数据类型   | 如果A等于NULL，则返回TRUE，反之返回FALSE                     |
| A IS NOT NULL           | 所有数据类型   | 如果A不等于NULL，则返回TRUE，反之返回FALSE                   |
| IN(数值1, 数值2)        | 所有数据类型   | 使用 IN运算显示列表中的值                                    |
| A [NOT] LIKE B          | STRING 类型    | B是一个SQL下的简单正则表达式，也叫通配符模式，如果A与其匹配的话，则返回TRUE；反之返回FALSE。B的表达式说明如下：‘x%’表示A必须以字母‘x’开头，‘%x’表示A必须以字母’x’结尾，而‘%x%’表示A包含有字母’x’,可以位于开头，结尾或者字符串中间。如果使用NOT关键字则可达到相反的效果。 |
| A RLIKE B, A REGEXP B   | STRING 类型    | B是基于java的正则表达式，如果A与其匹配，则返回TRUE；反之返回FALSE。匹配使用的是JDK中的正则表达式接口实现的，因为正则也依据其中的规则。例如，正则表达式必须和整个字符串A相匹配，而不是只需与其字符串匹配。 |

# 6. Like和RLike

> 1. % 代表0个或多个字符
> 2. _ 代表一个字符
> 3. RLIKE 子句是Hive中这个功能的一个扩展，其可以通过Java的正则表达式这个更强大的语言来指定匹配条件。

实例

（1）查找以2开头薪水的员工信息

```sql
select * from emp where sal LIKE '2%';
```

（2）查找第二个数值为2的薪水的员工信息

```sql
select * from emp where sal LIKE '_2%';
```

（3）查找薪水中含有2的员工信息

```sql
 select * from emp where sal RLIKE '[2]';
```

# 7. 逻辑运算符：和MySQL一致

| 操作符 | 含义   |
| ------ | ------ |
| AND    | 逻辑并 |
| OR     | 逻辑或 |
| NOT    | 逻辑否 |

# 8. 分组

## 1. group by

> GROUP BY语句通常会和聚合函数一起使用，按照一个或者多个列队结果进行分组，然后对每个组执行聚合操作。

实例

1. 计算emp表每个部门的平均工资

   ```sql
   select deptno,avg(sal) from emp;
   ```

2. 计算emp每个部门中每个岗位的最高薪水

   ```sql
   select deptno,job,max(sal) from emp group by deptno,job;
   ```

## 2. Having

1. Having与where的不同点

   > 1. where 后面不能写分组函数，而having可以
   > 2. having只用于group by分组统计语句

2. 实例

   求每个部门的平均薪水大于2000的部门

   ```sql
   -- 1. 每个部门的平均薪水 2.大于2000 
   select deptno, avg(sal) avg_sal from emp group by deptno having avg_sal > 2000;
   ```

   

# 9. join语句

## 1. 等值join

> Hive支持通常的SQL JOIN语句，但是**只支持等值连接，不支持非等值连接。**(和内连接可以理解为一样)

（1）根据员工表和部门表中的部门编号相等，查询员工编号、员工名称和部门名称；

```sql
select empno,ename,e.deptno from emp e join dept d on e.deptno = d.deptno;
```



## 2. 内连接(join)

> 只有进行连接的两个表中都存在与连接条件相匹配的数据才会被保留下来

## 3. 左外连接(left join)

> JOIN操作符左边表中符合WHERE子句的所有记录将会被返回，右边表没有的将会是（NULL值替代）

```sql
select empno,ename,e.deptno from emp e left join dept d on e.deptno = d.deptno;
```



## 4. 右外连接(right join)

> JOIN操作符右边表中符合WHERE子句的所有记录将会被返回。左边表没有的将会是（NULL值替代）

```sql
select empno,ename,e.deptno from emp e right join dept d on e.deptno = d.deptno;
```



## 5. 满外连接

> 将会返回所有表中符合WHERE语句条件的所有记录。如果任一表的指定字段没有符合条件的值的话，那么就使用NULL值替代。

```sql

```



## 6. 多表连接

> 大多数情况下，Hive会对每对JOIN连接对象启动一个MapReduce任务。本例中会首先启动一个MapReduce job对表e和表d进行连接操作，然后会再启动一个MapReduce job将第一个MapReduce job的输出和表l;进行连接操作。
>
> 优化：当对3个或者更多表进行join连接时，如果每个on子句都使用相同的连接键的话，那么只会产生一个MapReduce job。

```sql
SELECT e.ename, d.dname, l.loc_name
FROM   emp e 
JOIN   dept d
ON     d.deptno = e.deptno 
JOIN   location l
ON     d.loc = l.loc;
```



## 7. 笛卡尔积

> 产生笛卡尔积的条件：
>
> （1）省略连接条件
>
> （2）连接条件无效
>
> （3）所有表中的所有行互相连接

# 10. 排序

## 1. 全局排序（order by）

> Order By：全局排序，只有一个Reducer

## 2. 按别名或多列排序，与MySQL一致

## 3. sort by(每个MR内部排序)

> Sort By：对于大规模的数据集order by的效率非常低。在很多情况下，并不需要全局排序，此时可以使用**sort by**。
>
> Sort by**为每个reducer产生一个排序文件**。每个Reducer内部进行排序，对全局结果集来说不是排序。

实例

```sql
-- 1. 设置reduce个数
set mapreduce.job.reduces=3;
-- 2. 查看设置reduce个数
set mapreduce.job.reduces;
-- 3. 根据部门编号降序查看员工信息
select * from emp sort by deptno desc;
-- 4. 将查询结果导入到本地文件中（按照部门编号降序排序）
insert overwrite local directory '/home/mayi/sortby-result'
 select * from emp sort by deptno desc;
```

## 4. Distribute By（分区排序）

