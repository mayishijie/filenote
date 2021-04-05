# 0. [hive函数大全说明](https://www.iteblog.com/archives/2258.html)

# 1. 系统内置函数

## 1. 查询系统内置函数

```shell
hive (mayi)> show functions ;
```

## 2. 查看内置函数的使用方法

```shell
desc function explode;
# 或者
desc function extended explode;
```

# 2. 常用的内置函数

## 1. NVL：空字段赋值

> 给值为NULL的数据赋值，它的格式是NVL( value，default_value)。它的功能是如果value为NULL，则NVL函数返回default_value的值，否则返回value的值，如果两个参数都为NULL ，则返回NULL。

实例

```sql
-- 如果comm字段值是空，则用-1代替
select comm,nvl(comm,-1) from emp;
-- 如果comm字段值是空，则用领导id代替(意味着可以用其它字段值来填充)
select comm,nvl(comm,mrg) from emp;
```

## 2. case when：条件判断

语法

> case a when b then c [ when d then e]* [else f ] end
>
> when可以多个，else可选，end是结束

实例

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

## 3. 行转列（group by等函数组合）

> 1. CONCAT(string A/col, string B/col…)：返回输入字符串连接后的结果，支持任意个输入字符串;
>
> 2. CONCAT_WS(separator, str1, str2,...)：它是一个特殊形式的 CONCAT()。第一个参数剩余参数间的分隔符。分隔符可以是与剩余参数一样的字符串。如果分隔符是 NULL，返回值也将为 NULL。这个函数会跳过分隔符参数后的任何 NULL 和空字符串。分隔符将被加到被连接的字符串之间;
>
> 3. COLLECT_SET(col)：函数只接受基本数据类型，它的主要作用是将某字段的值进行去重汇总，产生array类型字段。

### 关键理解：

> 所谓行转列，关键几个函数组合使用：group by,concat,concat_ws,collect_set,collect_list
>
> 1. group by可以将多行数据按照指定字段分组
> 2. concat,concat_ws将行中的某些字段进行汇总拼接成字符串
> 3. collect_set,collect_list将行中的某些字段汇总到一个集合中

### 1. CONCAT:字符串连接函数

> 语法：concat(string A, string B…)
>
> 返回值：string

### 2. concat_ws:带分隔符字符串连接函数

> 语法：concat_ws(string SEP, string A, string B…)
>
> 返回值：string
>
> concat_ws(分隔符，字符串数组)

### 3. COLLECT_SET(col):将某字段的值进行去重汇总

> 除了将某个字段去重，我们有时候也会利用它变相的生产一个数组，有时候是为了使用这个数组
>
> 



### 实例

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



## 4. 列转行（EXPLODE）

> EXPLODE(col)：将hive一列中复杂的array或者map结构拆分成多行。
>
> LATERAL VIEW
>
> 用法：LATERAL VIEW udtf(expression) tableAlias AS columnAlias
>
> 解释：用于和split, explode等UDTF一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合。

实例

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

## 5. 窗口函数(开窗函数)

> OVER()：指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变而变化。
>
> CURRENT ROW (current row)：当前行
>
> n PRECEDING (n preceding)：往前n行数据
>
> n FOLLOWING (n following)：往后n行数据
>
> UNBOUNDED (unbounded)：起点，
>
> UNBOUNDED PRECEDING(unbounded preceding): 表示从前面的起点，
>
>  UNBOUNDED FOLLOWING(unbounded following)表示到后面的终点
>
> LAG(col,n,default_val)：往前第n行数据
>
> LEAD(col,n, default_val)：往后第n行数据
>
> NTILE(n)：把有序窗口的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，NTILE返回此行所属的组的编号。注意：n必须为int类型。

一下通过一些实例来说明窗口函数

数据:name，orderdate，cost

> jack,2017-01-01,10
>
> tony,2017-01-02,15
>
> jack,2017-02-03,23
>
> tony,2017-01-04,29
>
> jack,2017-01-05,46
>
> jack,2017-04-06,42
>
> tony,2017-01-07,50
>
> jack,2017-01-08,55
>
> mart,2017-04-08,62
>
> mart,2017-04-09,68
>
> neil,2017-05-10,12
>
> mart,2017-04-11,75
>
> neil,2017-06-12,80
>
> mart,2017-04-13,94

需求：

> （1）查询在2017年4月份购买过的顾客及总人数
>
> （2）查询顾客的购买明细及月购买总额
>
> （3）上述的场景, 将每个顾客的cost按照日期进行累加
>
> （4）查询每个顾客上次的购买时间
>
> （5）查询前20%时间的订单信息

实例

```sql
--数据准备
create table if not exists business
(
    name      string,
    orderdate string,
    cost      int
)
    row format delimited fields terminated by ',';
load data local inpath '/home/mayi/mayi_data/business.txt' 
into table business;
```

1.  查询在2017年4月份购买过的顾客及总人数

   ```sql
   select name,count(*) over ()
   from business
   where substring(orderdate,1,7) = '2017-04'
   group by name;
   -- 或者
   select distinct name, count(name) over()
   from business
   where  substring(orderdate, 1, 7) = '2017-04';
   
   -- 解析
   over()：窗口里面没有设置内容时，默认是全窗口，按照当前查询的窗口下内容进行count
   
   
   ```

   结果如下：

   ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44d642b9c6f146ae98d6d8c99bbd7150~tplv-k3u1fbpfcp-watermark.image)

   

2. 查询顾客的购买明细及月购买总额

   ```sql
   select name,orderdate,cost,
          sum(cost) over(partition by month(orderdate))
   from
    business;
    --分析
    over(partition by month(orderdate)):窗口根据里面月份分组进行月统计
   ```

   结果如下：

   ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3baa85d219964796aa631515808258a7~tplv-k3u1fbpfcp-watermark.image)

3. 上述的场景, 将每个顾客的cost按照日期进行累加

   ```sql
   select name,orderdate,cost,
          sum(cost) over(partition by month(orderdate)) mc,
          sum(cost) over(
              partition by name order by orderdate asc
              rows between unbounded preceding and current row
              ) lc
   from
    business;
    
   -- 解析
   partition by name order by orderdate asc： 按照name进行分组/分区，组内按照orderdate进行排序
   rows between unbounded preceding and current row： 
   	unbounded preceding：第一行
   	current row：当前行
   	
   	
   --实例
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

   结果如下：

   ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82e8443b658940f298807353c1ffe5d2~tplv-k3u1fbpfcp-watermark.image)

4. 查看顾客上次的购买时间

   ```sql
   select name,orderdate,cost,
          lag(orderdate,1) over (partition by name order by orderdate) last_order, --上一次购买时间
          lead(orderdate,1) over (partition by name order by orderdate) next_order -- 下一次购买时间
   from business;
   ```

   结果：

   ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9a8ee0dc2b34e3cb6e24f0e82725274~tplv-k3u1fbpfcp-watermark.image)

5. 查询前20%时间的订单信息

   ```sql
   -- 数据总共分5组，
   select name,orderdate,cost,
          ntile(5) over (order by orderdate) as group_name
   from business
   --前20%订单
   select *
   from (
            select name,
                   orderdate,
                   cost,
                   ntile(5) over (order by orderdate) as group_name
            from business
        ) t
   where t.group_name = 1;
   ```

   行组结果：

   ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23742d1e90d749029e5f9849cbd2536d~tplv-k3u1fbpfcp-watermark.image)

6. 求明细以及每个月有哪些顾客光顾过？

   ```sql
   select
    name,orderdate,cost,
     collect_set(name) over(partition by month(orderdate)) usr_list
   from business;
   
   -- 或
   select name,
          orderdate,
          cost,
          concat_ws(",", collect_set(name) over (partition by month(orderdate))) usr_list
   from business;
   ```

   结果：

   ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8078b9956f84d939a57475c5cfd229a~tplv-k3u1fbpfcp-watermark.image)

## 6. rank:排名

### 1. 函数说明

> RANK() 排序相同时会重复，总数不会变
>
> DENSE_RANK() 排序相同时会重复，总数会减少
>
> ROW_NUMBER() 会根据顺序计算

### 2. 实例

```sql
-- 数据准备
create table if not exists score(
    name string,
subject string, 
score int
)
row format delimited fields terminated by '\t';

load data local inpath '/home/mayi/mayi_data/score.txt'
into table score;

```

1. 每门学科的排名

   ```sql
   --每门的学科排名
   select *,
          rank() over (partition by subject order by score desc) rk,-- 排序相同会重复
          row_number() over (partition by subject order by score desc) rn, -- 会根据顺序计算
          dense_rank() over (partition by subject order by score desc) dr  -- 排序相同会重复，总数会减少
   from score;
   ```

   结果：

   ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/319669c99ae24254be06191e26a7fe0e~tplv-k3u1fbpfcp-watermark.image)

## 7. 日期函数

