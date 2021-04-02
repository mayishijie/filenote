# 0.可以参考如下项目，里面有大量bash相关的命令实例
[bash实例操作](https://github.com/wangdoc/bash-tutorial)
# 1. shell中的变量

## 1. 常用系统变量

> $HOME	:家目录
>
> $PWD	:当前路径
>
> $SHELL
>
> $USER

## 2. 自定义变量

> 1. 环境变量名建议大写
> 2. 等号2侧不能有空格
> 3. 变量的值如果有空格，必须用单引号或者双引号括起来
> 4. bash中默认类型都是字符串类型，无法直接进行数值运算

```shell
# 1. 定义一个变量a，并赋值未5
a=5
# 打印
echo $a

# 2.撤销,其实就是删除定义的变量
unset a

# 3. 定义静态变量readonly ,对于静态变量是不能利用unset撤销的
readonly a=5
```

## 3. 特殊变量

### 1. $n

> 1. n代表数字，$0代表脚本的名称
> 2. $1-9,代表第一参数到第九参数，
> 3. 10以及以上参数需要用大括号包裹起来，如：${10}

### 2. $#

> 代表输入参数的个数，一般用于循环

```shell
# 定义
#!/bin/bash
echo $#
# 执行
./test.sh a b c
# 结果
3
```

### 3. $*

> 将所有参数看做一个整体

```shell
# 还是上面的例子，里面添加定义 echo $*
# 执行结果：
a b c
```

### 4. $@

> 也是代表所有参数，只不过它是区分对待每个参数

```shell
# 还是上面的例子，里面添加定义 echo $@
# 执行结果：
a b c
```

### 5. $?

> 最后一次执行命令的返回状态，如果这个变量的值为0，证明上一个命令正确执行；如果这个变量的值为非0（具体是哪个数，由命令自己来决定），则证明上一个命令执行不正确了。

```shell
echo $?
#结果
0
```



# 2. 运算符

## 2.1. 基本语法

> 推荐用法：
>
> $((运算式))
>
> 或 $[运算式]
>
> 
>
> 这种个人用的比较少：
>
> 或 expr 运算式
>
> 注意：**利用expr需要注意，运算符之间必须要有空格**

## 2.2. 举例

```shell
# 定义变量
a=5
b=1
c=2
# 执行(a+b)*c的计算
echo $(((a+b)*2))
# 结果
6
# 或者
echo $[(a+b)*c]
# 或者
echo expr a + b

```

# 3. 条件判断

## 3.1. 常用的判断条件

### 3.1. 2个整数之间比较

> = 字符串比较
>
> -lt 小于（less than）			-le 小于等于（less equal）
>
> -eq 等于（equal）				-gt 大于（greater than）
>
> -ge 大于等于（greater equal）	-ne 不等于（Not equal）

### 3.1.2. 按照文件权限判断

> -r 有读的权限（read）			-w 有写的权限（write）
>
> -x 有执行的权限（execute）

### 3.1.3. 按照文件类型判断

> -f 文件存在并且是一个常规的文件（file）
>
> -e 文件存在（existence）		-d 文件存在并是一个目录（directory）



## 3.2 if判断

> 注意点：
>
> 1. [ 条件判断式 ]，中括号和条件判断式之间必须有空格
> 2. **if后要有空格**

### 3.2.1 语法

```shell
if [ 条件判断式 ] ;then
	程序逻辑
fi

或者

if [] 
	then
		程序逻辑
elif []
	then 
		程序逻辑
else
	程序逻辑
fi
```

### 3.2.2 举例

```shell#!/bin/bash
#!/bin/bash
if [ $1 == 1 ]
	then
		echo 'fisrt'
elif [ $1 == 2 ]
	then
		echo 'seconde'
else
	echo '333333'
fi
```

## 3.4 case语句

### 3.4.1语法

> case $变量名 in 
>
>  "值1"） 
>
>   如果变量的值等于值1，则执行程序1 
>
>   ;; 
>
>  "值2"） 
>
>   如果变量的值等于值2，则执行程序2 
>
>   ;; 
>
>  …省略其他分支… 
>
>  *） 
>
>   如果变量的值都不是以上的值，则执行此程序 
>
>   ;; 
>
> esac

**注意**

> 注意事项：
>
> 1) case行尾必须为单词“in”，每一个模式匹配必须以右括号“）”结束。
>
> 2) 双分号“;;”表示命令序列结束，相当于java中的break。
>
> 3) 最后的“*）”表示默认模式，相当于java中的default。

### 3.4.2 举例

```shell
#!/bin/bash
case $1 in
1)
echo '1111'
;;
2)
echo '2222'
;;
*)
echo 'other'
;;
esac
```

## 3.5 for循环

### 3.5.1 语法

> **方式1：**
>
> for (( 初始值;循环控制条件;变量变化 )) 
>
>  do 
>
>   程序 
>
>  done
>
> 
>
> **方式2：**
>
> for 变量 in 值1 值2 值3… 
>
>  do 
>
>   程序 
>
>  done

实例

```shell
#!/bin/bash

s=0
for((i=0;i<=100;i++))
do
        s=$[$s+$i]
done
echo $s
```



## 3.6 \$* 与 \$@对比

> 1. \$* \$@都表示传递给函数或脚本的所有参数，不被双引号“”包含时，都以$1 $2 …$n的形式输出所有参数。

实例：

```shell
#!/bin/bash 

for i in $*
do
      echo "ban zhang love $i "
done

for j in $@
do      
        echo "ban zhang love $j"
done

[mayi@mayi101 datas]$ bash for.sh cls xz bd
ban zhang love cls 
ban zhang love xz 
ban zhang love bd 
ban zhang love cls
ban zhang love xz
ban zhang love bd
```

> 2. 当它们被双引号“”包含时，“\$*”会将所有的参数作为一个整体，以“$1 $2 …​\$n”的形式输出所有参数；“\$@”会将各个参数分开，以“$1” “$2”…”$n”的形式输出所有参数。

实例

```shell
#!/bin/bash 

for i in "$*" 
#$*中的所有参数看成是一个整体，所以这个for循环只会循环一次 
        do 
                echo "ban zhang love $i"
        done 

for j in "$@" 
#$@中的每个参数都看成是独立的，所以“$@”中有几个参数，就会循环几次 
        do 
                echo "ban zhang love $j" 
done
```

## 3.7 while循环

> while [ 条件判断式 ] 
>
>  do 
>
>   程序
>
>  done

实例

```shell
#!/bin/bash
s=0
i=1
while [ $i -le 100 ]
do
        s=$[$s+$i]
        i=$[$i+1]
done

echo $s
```

# 4. 函数

## 4.1 系统函数

### 4.1.1 basename:删除路径取文件名

> basename [string / pathname] [suffix]  （功能描述：basename命令会删掉所有的前缀包括最后一个（‘/’）字符，然后将字符串显示出来。
>
> 选项：
>
> suffix为后缀，如果suffix被指定了，basename会将pathname或string中的suffix去掉。

实例

```shell
# 删除前缀
➜  filenote git:(master) ✗ basename /Users/tty/IdeaProjects/bigdata/filenote/test.sh
test.sh
# 删除前缀以及后缀
filenote git:(master) ✗ basename /Users/tty/IdeaProjects/bigdata/filenote/test.sh .sh
test
```

### 4.1.2 dirname:删除文件去路径名

> dirname 文件绝对路径		（功能描述：从给定的包含绝对路径的文件名中去除文件名（非目录的部分），然后返回剩下的路径（目录的部分））

实例

```shell
#
➜  filenote git:(master) ✗ dirname /Users/tty/IdeaProjects/bigdata/filenote/test.sh
/Users/tty/IdeaProjects/bigdata/filenote

```

## 4.2 自定义函数

### 1. 语法

> [ function ] funname[()]
>
> {
>
> ​	Action;
>
> ​	[return int;]
>
> }
>
> funname

### 2. 经验技巧

> （1）必须在调用函数地方之前，先声明函数，shell脚本是逐行运行。不会像其它语言一样先编译。
>
> （2）函数返回值，只能通过$?系统变量获得，可以显示加：return返回，如果不加，将以最后一条命令运行结果，作为返回值。return后跟数值n(0-255)

### 3. 实例

```shell
#!/bin/bash
function sum()
{
    s=0
    s=$[ $1 + $2 ]
    echo "$s"
}
```

# 5. shell工具

## 1. cut

> cut的工作就是“剪”，具体的说就是在文件中负责剪切数据用的。cut 命令从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段输出

### 基本用法

>  cut [选项参数]  filename
>
> 说明：默认分隔符是制表符

### 参数选项

| 选项参数 | 功能                         |
| -------- | ---------------------------- |
| -f       | 列号，提取第几列             |
| -d       | 分隔符，按照指定分隔符分割列 |
| -c       | 指定具体的字符               |

实例：
```shell
# 选取系统PATH变量值，第2个“：”开始后的所有路径：
➜  filenote git:(master) ✗ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Applications/VMware Fusion.app/Contents/Public

➜  filenote git:(master) ✗ echo $PATH|cut -d : -f 2-
```

## 2. sed

> sed是一种流编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。**文件内容并没有改变**，除非你使用重定向存储输出。

### 基本用法

> sed [选项参数]  ‘command’  filename

选项参数

| 选项参数 | 功能                                  |
| -------- | ------------------------------------- |
| -e       | 直接在指令列模式上进行sed的动作编辑。 |
| -i       | 直接编辑文件                          |

命令功能描述

| 命令  | 功能描述                              |
| ----- | ------------------------------------- |
| **a** | 新增，a的后面可以接字串，在下一行出现 |
| d     | 删除                                  |
| s     | 查找并替换                            |

## 3. awk

## 4. sort

