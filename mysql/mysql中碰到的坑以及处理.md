# 1. mysql中大小写敏感问题

[参考大小写敏感](https://juejin.cn/post/6844903864483708941)

> 问题背景
>
> 今天创建了一张表，但是在工具上查询时，却查不到这张表。
>
> ```mysql
> select * from staff; --  可以查出
> select * from STAFF; -- 显示没有该表
> 
> -- 查询库下表信息
> show tables;
> staff
> ```
>
> 通过上面的查询看到，明明有staff表，但是一变成大写，就说没有这个表了，基本可以推断出它这里大小写敏感了，后面去查询相关资料说，MySQL在window上时，对大小写是不敏感的，但是在Linux服务器上时敏感的。
>
> 
>
> 1. MySQL在[Linux](http://lib.csdn.net/base/linux)下数据库名、表名、列名、别名大小写规则是这样的： 
>    　　1、数据库名与表名是严格区分大小写的； 
>    　　2、表的别名是严格区分大小写的； 
>    　　3、列名与列的别名在所有的情况下均是忽略大小写的； 
>    　　4、变量名也是严格区分大小写的； 
> 2. MySQL在Windows下都不区分大小写。
>
> 
>
> 碰到问题肯定得解决问题，后面继续找资料，通过如下命令可以查看当前系统是否大小写敏感
>
> ```sql
> show variables like ‘%case%’; 
> --结果
> lower_case_file_system,OFF	：-- off表明大小写敏感
> lower_case_table_names,1	：-- 默认值是0，大小写敏感，1或者2表明不敏感
> ```
>
> 修改MySQL的配置，让它不敏感大小写：
>
> 1. 修改lower_case_table_names=1，需要修改/etc/my.cnf文件，在该文件中添加一行lower_case_table_names=1
>
> 查看my.cnf文件路径：mysql --help | grep 'my.cnf'
>
> 2. 重启MySQL服务，一下列出常用的Linux服务相关命令
>
>    ```shell
>    # mysql 状态查看
>    systemctl status mysql
>    # 启动MySQL服务
>    systemctl start mysqld
>    # 其它命令（vsftpd就是指需要启动的服务）
>    systemctl start/sop/status/restart vsftpd
>    ```
>
>    
>
> 
