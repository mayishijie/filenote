# 1. sqoop导入和导出空值问题？

> 分析：对于空值，在MySQL是null形式保存，就是空值，但是在hive中，空值是以\N保存，因为这两者不一致，会导致将数据导入到hive中，null值会作为字符串存在，而这个些情况是应该避免的。

## import 导入（MySQL-> Hive）：

2个关键的参数去空值：

> 1. --null-string '@@'  如果是字符串类型，且值是空值，则用字符串@@来代替value值，具体值根据个人项目实际情况定
> 2. --null-non-string '##' 如果是非字符串类型，且值是空值，那么用###来替代value值

## export 导出（Hive->Mysql）:

> 1. --input-null-string：含义是和--null-string一致
> 2. --input-null-non-string ：含义是和--null-non-string一致

# 2. sqoop数据导出一致性问题？

## 1. 产生问题？

> 由于Sqoop将导出过程分解为多个事务，因此失败的导出作业可能会导致部分数据提交到数据库。在某些情况下，这可能进一步导致后续作业因插入冲突而失败，而在其他情况下，则可能导致数据重复。您可以通过--staging-table选项指定暂存表来克服此问题，该选项用作用于暂存导出数据的辅助表。最后，已分阶段处理的数据将在单个事务中移至目标表。

## 2. 解决办法

> | --staging-table <staging-table-name> | 创建一张临时表，用于存放所有事务的结果，然后将所有事务结果一次性导入到目标表中，防止错误。 |
> | ------------------------------------ | ------------------------------------------------------------ |
> | --clear-staging-table                | 如果第--stageing-table参数非空，则可以在导出操作执行前，清空临时事务结果表 |

# 3. sqoop导入数据发送数据倾斜？

[参考如下文章](https://blog.csdn.net/lizhiguo18/article/details/103969906)

> 在了解数据切分策略之后，在数据表没有符合要求的字段的情况下，我们需要自己临时简单创建理想的字段。
>
> 这里需要使用--query 方式：涉及参数  --query、--split-by、--boundary-query
>
> --query: select column1、 column2、 column3、 columnN from (select ROWNUM() OVER() AS ETL_ID, T.* from table T where xxx )  where $CONDITIONS
>
> --split-by: ETL_ID
>
> --boundary-query: select 1 as MIN , sum(1) as MAX from table where xxx
>
> 具体原理就是通过ROWNUM() 生成一个严格均匀分布的字段，然后指定为分割字段。

# 4. sqoop原理

​		默认是4个map，只有map阶段,没有reduce阶段

> 1. 将导入或导出命令翻译成mapreduce程序来实现。
> 2. 在翻译出的mapreduce中主要是对inputformat和outputformat进行定制。