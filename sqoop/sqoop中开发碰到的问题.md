# 0. 导入和导出概念

> Sqoop中的导入，是指从关系型数据库中数据导入到HDFS等大数据平台，反之则是导出

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

# 5. sqoop开发需要注意点：

## 1. 查询导入时，必须要包含$CONDITIONS

> $CONDITIONS 是数据分割占位符，必须存在，当query后面是用双引号括起来，需要\$CONDITIONS前面用转义符，不然会被shell识别为自己的变量

```sql
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--query 'select name,sex from staff where id <=1 and $CONDITIONS;'
```

## 2. 导入指定列

> columns中如果涉及到多列，用逗号分隔，分隔时不要添加空格

```sql
--columns id,sex \
```

# 6. MySql->HDFS

## 1. 数据准备

```sql
mysql> create database company;
mysql> create table company.staff(id int(4) primary key not null auto_increment, name varchar(255), sex varchar(255));
mysql> insert into company.staff(name, sex) values('Thomas', 'Male');
mysql> insert into company.staff(name, sex) values('Catalina', 'FeMale');
```

## 2. 全部导入

```sql
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t"
```

## 3. 查询导入

```sql
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--query 'select name,sex from staff where id <=1 and $CONDITIONS;'
```

## 4. 导入指定列

```sql
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--columns id,sex \
--table staff
```

> columns中如果涉及到多列，用逗号分隔，分隔时不要添加空格

## 5. 使用sqoop关键字筛选导入

```sql
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--table staff \
--where "id=1"
```



# 7. MySql -> Hive

> 提示：该过程分为两步，第一步将数据导入到HDFS，第二步将导入到HDFS的数据迁移到Hive仓库，第一步默认的临时目录是/user/mayi/表名

```sql
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--num-mappers 1 \
--hive-import \
--fields-terminated-by "\t" \
--hive-overwrite \
--hive-table staff_hive
```



# 8. Mysql -> HBase

> sqoop1.4.6只支持HBase1.0.1之前的版本的自动创建HBase表的功能
>
> 手动创建
>
> hbase> create 'hbase_company','info'

```sql
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--columns "id,name,sex" \
--column-family "info" \
--hbase-create-table \
--hbase-row-key "id" \
--hbase-table "hbase_company" \
--num-mappers 1 \
--split-by id
```

# 9. Hive/HDFS -> MySql

> 如果MySQL中没有创建该staff表，不会自动创建，所以导入前，先自己手动创建

```sql
$ bin/sqoop export \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--num-mappers 1 \
--export-dir /user/hive/warehouse/staff_hive \
--input-fields-terminated-by "\t"
```

# 10. sqoop脚本打包

> 使用opt格式的文件打包sqoop命令，然后执行

## 1. 创建.opt文件

```shell
 mkdir opt
 touch opt/job_HDFS2RDBMS.opt
```

## 2. 编写脚本内容

```shell
export
--connect
jdbc:mysql://hadoop102:3306/company
--username
root
--password
000000
--table
staff
--num-mappers
1
--export-dir
/user/hive/warehouse/staff_hive
--input-fields-terminated-by
"\t"
```

## 3. 脚本执行

```shell
bin/sqoop --options-file opt/job_HDFS2RDBMS.opt
```

# 11.sqoop常用命令列举

## 1. 常用命令

| 序号 | 命令              | 类                  | 说明                                                         |
| ---- | ----------------- | ------------------- | ------------------------------------------------------------ |
| 1    | import            | ImportTool          | 将数据导入到集群                                             |
| 2    | export            | ExportTool          | 将集群数据导出                                               |
| 3    | codegen           | CodeGenTool         | 获取数据库中某张表数据生成Java并打包Jar                      |
| 4    | create-hive-table | CreateHiveTableTool | 创建Hive表                                                   |
| 5    | eval              | EvalSqlTool         | 查看SQL执行结果                                              |
| 6    | import-all-tables | ImportAllTablesTool | 导入某个数据库下所有表到HDFS中                               |
| 7    | job               | JobTool             | 用来生成一个sqoop的任务，生成后，该任务并不执行，除非使用命令执行该任务。 |
| 8    | list-databases    | ListDatabasesTool   | 列出所有数据库名                                             |
| 9    | list-tables       | ListTablesTool      | 列出某个数据库下所有表                                       |
| 10   | merge             | MergeTool           | 将HDFS中不同目录下面的数据合在一起，并存放在指定的目录中     |
| 11   | metastore         | MetastoreTool       | 记录sqoop job的元数据信息，如果不启动metastore实例，则默认的元数据存储目录为：~/.sqoop，如果要更改存储目录，可以在配置文件sqoop-site.xml中进行更改。 |
| 12   | help              | HelpTool            | 打印sqoop帮助信息                                            |
| 13   | version           | VersionTool         | 打印sqoop版本信息                                            |

## 2. 命令&参数详解

### 1. 公用参数：数据库连接

| ***\*序号\**** | ***\*参数\****       | ***\*说明\****         |
| -------------- | -------------------- | ---------------------- |
| 1              | --connect            | 连接关系型数据库的URL  |
| 2              | --connection-manager | 指定要使用的连接管理类 |
| 3              | --driver             | Hadoop根目录           |
| 4              | --help               | 打印帮助信息           |
| 5              | --password           | 连接数据库的密码       |
| 6              | --username           | 连接数据库的用户名     |
| 7              | --verbose            | 在控制台打印出详细信息 |

### 2. 公用参数：import导入

| 1    | --enclosed-by <char>            | 给字段值前加上指定的字符                                     |
| ---- | ------------------------------- | ------------------------------------------------------------ |
| 2    | --escaped-by <char>             | 对字段中的双引号加转义符                                     |
| 3    | --fields-terminated-by <char>   | 设定每个字段是以什么符号作为结束，默认为逗号                 |
| 4    | --lines-terminated-by <char>    | 设定每行记录之间的分隔符，默认是\n                           |
| 5    | --mysql-delimiters              | Mysql默认的分隔符设置，字段之间以逗号分隔，行之间以\n分隔，默认转义符是\，字段值以单引号包裹。 |
| 6    | --optionally-enclosed-by <char> | 给带有双引号或单引号的字段值前后加上指定字符。               |

### 3. 公用参数：export导出

| 1    | --input-enclosed-by <char>            | 对字段值前后加上指定字符                   |
| ---- | ------------------------------------- | ------------------------------------------ |
| 2    | --input-escaped-by <char>             | 对含有转移符的字段做转义处理               |
| 3    | --input-fields-terminated-by <char>   | 字段之间的分隔符                           |
| 4    | --input-lines-terminated-by <char>    | 行之间的分隔符                             |
| 5    | --input-optionally-enclosed-by <char> | 给带有双引号或单引号的字段前后加上指定字符 |

### 4. 公用参数：hive

| 1    | --hive-delims-replacement <arg> | 用自定义的字符串替换掉数据中的\r\n和\013 \010等字符       |
| ---- | ------------------------------- | --------------------------------------------------------- |
| 2    | --hive-drop-import-delims       | 在导入数据到hive时，去掉数据中的\r\n\013\010这样的字符    |
| 3    | --map-column-hive <arg>         | 生成hive表时，可以更改生成字段的数据类型                  |
| 4    | --hive-partition-key            | 创建分区，后面直接跟分区名，分区字段的默认类型为string    |
| 5    | --hive-partition-value <v>      | 导入数据时，指定某个分区的值                              |
| 6    | --hive-home <dir>               | hive的安装目录，可以通过该参数覆盖之前默认配置的目录      |
| 7    | --hive-import                   | 将数据从关系数据库中导入到hive表中                        |
| 8    | --hive-overwrite                | 覆盖掉在hive表中已经存在的数据                            |
| 9    | --create-hive-table             | 默认是false，即，如果目标表已经存在了，那么创建任务失败。 |
| 10   | --hive-table                    | 后面接要创建的hive表,默认使用MySQL的表名                  |
| 11   | --table                         | 指定关系数据库的表名                                      |

