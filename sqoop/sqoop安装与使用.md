# 1. Sqoop简介与原理

1. 简介

   > sqoop是一个用于Hadoop(hive)与传统数据库(mysql/oracle等)之间进行数据相互导入或者导出的工具

2. 原理

   > 将导入与导出命令翻译成mr程序来实现，在翻译的mr中，主要是对inputformat与outputformat进行定制。

# 2. sqoop安装

1. 上传压缩包并解压

   ```shell
   tar -zxf sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz -C /opt/module/
   ```

   

2. 修改配置文件

   ```shell
   # 1. 修改配置文件
   [mayi@mayi101 conf]$ mv sqoop-env-template.sh sqoop-env.sh
   # 2. 修改配置sqoop-env.sh,添加配置信息
   export HADOOP_COMMON_HOME=/opt/software/hadoop-2.9.2
   #Set path to where hadoop-*-core.jar is available
   export HADOOP_MAPRED_HOME=/opt/software/hadoop-2.9.2
   #set the path to where bin/hbase is available
   export HBASE_HOME=/opt/software/hbase-1.3.1
   #Set the path to where bin/hive is available#export HIVE_HOME=
   export HIVE_HOME=/opt/software/apache-hive-1.2.2-bin
   #Set the path for where zookeper config dir is
   export ZOOCFGDIR=/opt/software/zookeeper-3.4.10
   export ZOOKEEPER_HOME=/opt/software/zookeeper-3.4.10
   ```

   

3. 拷贝jdbc驱动

   ```shell
    cp -v mysql-connector-java-5.1.27-bin.jar /opt/software/sqoop1.4.6/lib/
   ```

   

4. 验证sqoop

   ```shell
   [mayi@mayi101 sqoop1.4.6]$ bin/sqoop help
   Warning: /opt/software/sqoop1.4.6/bin/../../hcatalog does not exist! HCatalog jobs will fail.
   Please set $HCAT_HOME to the root of your HCatalog installation.
   Warning: /opt/software/sqoop1.4.6/bin/../../accumulo does not exist! Accumulo imports will fail.
   Please set $ACCUMULO_HOME to the root of your Accumulo installation.
   SLF4J: Class path contains multiple SLF4J bindings.
   SLF4J: Found binding in [jar:file:/opt/software/hadoop-2.9.2/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
   SLF4J: Found binding in [jar:file:/opt/software/hbase-1.3.1/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
   SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
   SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
   21/04/07 06:54:24 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6
   usage: sqoop COMMAND [ARGS]
   
   Available commands:
     codegen            Generate code to interact with database records
     create-hive-table  Import a table definition into Hive
     eval               Evaluate a SQL statement and display the results
     export             Export an HDFS directory to a database table
     help               List available commands
     import             Import a table from a database to HDFS
     import-all-tables  Import tables from a database to HDFS
     import-mainframe   Import datasets from a mainframe server to HDFS
     job                Work with saved jobs
     list-databases     List available databases on a server
     list-tables        List available tables in a database
     merge              Merge results of incremental imports
     metastore          Run a standalone Sqoop metastore
     version            Display version information
   
   See 'sqoop help COMMAND' for information on a specific command.
   ```

   

5. 测试sqoop能否正常连接数据库

   ```shell
   [mayi@mayi101 sqoop1.4.6]$ bin/sqoop list-databases --connect  jdbc:mysql://mayi101:3306/ --username root --password root
   Warning: /opt/software/sqoop1.4.6/bin/../../hcatalog does not exist! HCatalog jobs will fail.
   Please set $HCAT_HOME to the root of your HCatalog installation.
   Warning: /opt/software/sqoop1.4.6/bin/../../accumulo does not exist! Accumulo imports will fail.
   Please set $ACCUMULO_HOME to the root of your Accumulo installation.
   SLF4J: Class path contains multiple SLF4J bindings.
   SLF4J: Found binding in [jar:file:/opt/software/hadoop-2.9.2/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
   SLF4J: Found binding in [jar:file:/opt/software/hbase-1.3.1/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
   SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
   SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
   21/04/07 07:02:09 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6
   21/04/07 07:02:10 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
   21/04/07 07:02:10 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
   information_schema
   hive
   mayi
   mysql
   performance_schema
   sys
   ```

   

# 3. sqoop简单使用

## 1. 导入数据

> 在Sqoop中，“导入”概念指：从非大数据集群（RDBMS）向大数据集群（HDFS，HIVE，HBASE）中传输数据，叫做：导入，即使用import关键字。

### 1. RDBMS到hdfs



### 2. RDBMS到hive

### 3. RDBMS到HBASE

## 2. 导出数据

### hive/hdfs到RDBMS

## 3. 脚本打包（opt）

# 4. sqoop常用命令以及参数

## 1. 常用命令

| **序号** | **命令**          | **类**              | **说明**                                                     |
| -------- | ----------------- | ------------------- | ------------------------------------------------------------ |
| 1        | import            | ImportTool          | 将数据导入到集群                                             |
| 2        | export            | ExportTool          | 将集群数据导出                                               |
| 3        | codegen           | CodeGenTool         | 获取数据库中某张表数据生成Java并打包Jar                      |
| 4        | create-hive-table | CreateHiveTableTool | 创建Hive表                                                   |
| 5        | eval              | EvalSqlTool         | 查看SQL执行结果                                              |
| 6        | import-all-tables | ImportAllTablesTool | 导入某个数据库下所有表到HDFS中                               |
| 7        | job               | JobTool             | 用来生成一个sqoop的任务，生成后，该任务并不执行，除非使用命令执行该任务。 |
| 8        | list-databases    | ListDatabasesTool   | 列出所有数据库名                                             |
| 9        | list-tables       | ListTablesTool      | 列出某个数据库下所有表                                       |
| 10       | merge             | MergeTool           | 将HDFS中不同目录下面的数据合在一起，并存放在指定的目录中     |
| 11       | metastore         | MetastoreTool       | 记录sqoop job的元数据信息，如果不启动metastore实例，则默认的元数据存储目录为：~/.sqoop，如果要更改存储目录，可以在配置文件sqoop-site.xml中进行更改。 |
| 12       | help              | HelpTool            | 打印sqoop帮助信息                                            |
| 13       | version           | VersionTool         | 打印sqoop版本信息                                            |

## 2. 命令/参数详解

> 刚才列举了一些Sqoop的常用命令，对于不同的命令，有不同的参数，让我们来一一列举说明。
>
> 首先来我们来介绍一下公用的参数，所谓公用参数，就是大多数命令都支持的参数。

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

### 2. 公用参数：import

| ***\*序号\**** | ***\*参数\****                  | ***\*说明\****                                               |
| -------------- | ------------------------------- | ------------------------------------------------------------ |
| 1              | --enclosed-by <char>            | 给字段值前加上指定的字符                                     |
| 2              | --escaped-by <char>             | 对字段中的双引号加转义符                                     |
| 3              | --fields-terminated-by <char>   | 设定每个字段是以什么符号作为结束，默认为逗号                 |
| 4              | --lines-terminated-by <char>    | 设定每行记录之间的分隔符，默认是\n                           |
| 5              | --mysql-delimiters              | Mysql默认的分隔符设置，字段之间以逗号分隔，行之间以\n分隔，默认转义符是\，字段值以单引号包裹。 |
| 6              | --optionally-enclosed-by <char> | 给带有双引号或单引号的字段值前后加上指定字符。               |

### 3. 公用参数：export

| ***\*序号\**** | ***\*参数\****                        | ***\*说明\****                             |
| -------------- | ------------------------------------- | ------------------------------------------ |
| 1              | --input-enclosed-by <char>            | 对字段值前后加上指定字符                   |
| 2              | --input-escaped-by <char>             | 对含有转移符的字段做转义处理               |
| 3              | --input-fields-terminated-by <char>   | 字段之间的分隔符                           |
| 4              | --input-lines-terminated-by <char>    | 行之间的分隔符                             |
| 5              | --input-optionally-enclosed-by <char> | 给带有双引号或单引号的字段前后加上指定字符 |

### 4. 公用参数：hive

| ***\*序号\**** | ***\*参数\****                  | ***\*说明\****                                            |
| -------------- | ------------------------------- | --------------------------------------------------------- |
| 1              | --hive-delims-replacement <arg> | 用自定义的字符串替换掉数据中的\r\n和\013 \010等字符       |
| 2              | --hive-drop-import-delims       | 在导入数据到hive时，去掉数据中的\r\n\013\010这样的字符    |
| 3              | --map-column-hive <arg>         | 生成hive表时，可以更改生成字段的数据类型                  |
| 4              | --hive-partition-key            | 创建分区，后面直接跟分区名，分区字段的默认类型为string    |
| 5              | --hive-partition-value <v>      | 导入数据时，指定某个分区的值                              |
| 6              | --hive-home <dir>               | hive的安装目录，可以通过该参数覆盖之前默认配置的目录      |
| 7              | --hive-import                   | 将数据从关系数据库中导入到hive表中                        |
| 8              | --hive-overwrite                | 覆盖掉在hive表中已经存在的数据                            |
| 9              | --create-hive-table             | 默认是false，即，如果目标表已经存在了，那么创建任务失败。 |
| 10             | --hive-table                    | 后面接要创建的hive表,默认使用MySQL的表名                  |
| 11             | --table                         | 指定关系数据库的表名                                      |



###  命令参数下面

### 5. 