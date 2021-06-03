基础数据准备

```text
# name date cost
jack,2017-01-01,10
tony,2017-01-02,15
jack,2017-02-03,23
tony,2017-01-04,29
jack,2017-01-05,46
jack,2017-04-06,42
tony,2017-01-07,50
jack,2017-01-08,55
mart,2017-04-08,62
mart,2017-04-09,68
neil,2017-05-10,12
mart,2017-04-11,75
neil,2017-06-12,80
mart,2017-04-13,94
```

# 窗口函数说明

> over()：指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变而变化。
>
> current row：当前行
>
> n preceding: 往前n行数据
>
> n following：往后n行数据
>
> unbounded：起点，
>
> unbounded preceding ：表示从前面的起点(可以理解为起点的每一行)
>
> unbounded following：表示到后面的终点
>
> lag(col,n,default_val)：往前第n行数据
>
> lead(col,n, default_val)：往后第n行数据
>
> ntile(n)：把有序窗口的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，NTILE返回此行所属的组的编号。注意：n必须为int类型。

# 1. 查询在2017年4月份购买过的顾客及总人数

```sql
select name,count(*) over () 
from business 
where substring(orderdate,1,7) = '2017-04' 
group by name;

-- 上面的解析步骤可以理解为如下
-- 1. 先查询名字
select name
from business 
where substring(orderdate,1,7) = '2017-04' 
group by name;
-- 2. 对步骤1的结果开窗，over没有参数，那么窗口就是整个上面查询结果，可以理解为一张临时虚拟表
select name,count(*) over () 
from business 
where substring(orderdate,1,7) = '2017-04' 
group by name;
-- 3. count(*),既然已经开窗是整个上面查询结果，那么此刻count就是整个窗口下的所有数据
```

## 解析：

> over()里面没有任何参数，按照上面的例子，那窗口就是整个查询结果对应的一个表，可以理解为就是一个虚拟表，然后在这个虚拟表中，统计指定字段数量
>
> over其实就是指定聚合的范围，为那个count函数 

# 2. 查询顾客购买明细和月购买总额

```sql
select name,orderdate,cost,
sum(cost) over(partition by month(orderdate))
from business;
-- 1. 查询明细
-- 2. over对临时表中日期开窗，按月分组partition by month(orderdate)
-- 3. sum(cost) 对开窗分组后的数据统计总数
```

# 3. 上述的场景, 将每个顾客的cost按照日期进行累加

```sql
select name,
       orderdate,
       cost,
       sum(cost) over (partition by month(orderdate)) as ms,
       sum(cost) 
       over (partition by name order by orderdate asc rows between unbounded preceding and current row )       				as leijiasum
from business;

--  这个leijiasum，主要分析这个sum(cost)
-- 1. partition by name 按照名字分组（相当于有个临时表）
-- 2. order by orderdate asc 这个临时表的数据按照时间进行升序排序
-- 3. rows between 统计的行数范围
-- 4. unbounded preceding and current row  
```

# 4. 窗口小结实例

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

# 5. 查看顾客上次的购买时间	LAG使用

```sql
select name,
       orderdate,
       cost,
      lag(orderdate,1,'1907-01-01')
           over (partition by name order by orderdate asc) as leijiasum
from business;

-- lag函数：显示某一列数据的上几行，窗口有序函数
-- lag 上几行
-- lead 下几行
```

# 6. 查询前20%时间的订单信息 ntile

