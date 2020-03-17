# Linux 之Shell脚本

## 1. Shell变量

变量：是shell传递数据的一种方式，用来代表每个取值的符号名。当shell脚本需要保存一些信息时，如一个文件名或是一个数字，就把它存放在一个变量中。 

### 1.1 变量设置规则

  　　1. 变量名称可以由**字母，数字和下划线**组成，但是不能以数字开头，**环境变量名建议大写**，便于区分。
        2. 在bash中，变量的**默认类型都是字符串型**，如果要进行数值运算，则必须**指定变量类型为数值型**。
        3. 变量用**等号**连接值，等号左右两侧**不能有空格**。
        4. 变量的值如果有空格，需要使用单引号或者双引号包括。

```shell
STR="hello shell" #√
STR = "hello shell" #×, =号两边不能出现空格
```

### 1.2 变量分类

分为用户自定义变量d环境变量，位置参数变量和预定义变量。 可以通过set命令查看系统中存在的所有变量。 

- 系统变量：保存和系统操作环境相关的数据。HOME、PWD、SHELL、USER 等等
- 位置参数变量：主要用来向脚本中传递参数或数据，变量名不能自定义，变量作用固定。
- 预定义变量：是Bash中已经定义好的变量，变量名不能自定义，变量作用也是固定的。
- 用户自定义变量：用户自定义的变量由字母或下划线开头，由字母，数字或下划线序列组成。

### 1.3 变量调用

 在使用变量时，要在变量名前加上前缀`$`。

```shell
# 设置变量
STR="hello shell"
echo $STR
echo ${STR} # or
```

> [!NOTE]
>
> 推荐给所有的变量加上`{}`
>
>  **$()****与${}的区别** :  $( )的用途和反引号``一样，用来表示优先执行的命令 

#### 1.3.1 Set

```shell
set #列出所有变量
unset A 
```

#### 1.3.2 export

 如果把环境变量写入相应的配置文件，那么这个环境变量就会在所有的shell中生效。 

```shell
export 变量名=变量值
```

#### 1.3.3 位置参数变量

|变量|描述|
| ---- | :----------------------------------------------------------- |
| $n   | n为数字，0代表命令本身，0代表命令本身，1-$9代表第一到第9个参数,十以上的参数需要用大括号包含，如${10}。 |
| $*   | 输出所有参数值。且它们为一个整体。不可分割。 |
| $@   | 输出所有参数值。用空格分开。每个参数都是独立的。 |
| $#   | 参数个数 |



#### 1.3.4 预定义变量
|变量|描述|
| ---- | ------------------------------------------------------------ |
| $?   | 执行上一个命令的返回值 执行成功，返回0，执行失败，返回非0（具体数字由命令决定） |
| $$   | 当前进程的进程号（PID），即当前脚本执行时生成的进程号        |
| $!   | 后台运行的最后一个进程的进程号（PID），最近一个被放入后台执行的进程  & |

## 2. 运算符

### 2.1 expr

 对整数型变量进行算术运算 。

> [!NOTE]
>
>  运算符前后必须要有空格。

```shell
S=`expr 3 + 3`
echo ${S}

S=`expr 2 + 3`
echo `expr ${S} \* 4`
```

> [!NOTE]
>
> 1. 算术表达式写在``里面。且表达式用空格分开。`
> 2. *`需要使用`\`转义。

### 2.2 test

 置test命令常用操作符号`[]`表示。

```shell
[ expression ]
```

测试范围：整数、字符串、文件。

| 表达式结果 | test返回值 | $?   |
| ---------- | ---------- | ---- |
| 真         | 0          | 0    |
| 假         | 非0        | 非0  |

#### 2.3.1 字符串测试

| 测试类型          | 结果                          |
| ----------------- | ----------------------------- |
| test str1 == str2 | 字符串是否相等                |
| test str1 != str2 | 字符串是否不相等              |
| test str1         | 字符串是否不为空。不为空:true |
| test -n str1      | 测试字符串是否不为空          |
| test -z str1      | 测试字符串是否为空            |

```shell
# 类似于三元运算符
name=lisi
test "${name}" == "lisi" && echo ok || echo invalid
```

#### 2.3.2 整数测试

| 测试类型            | 结果                    | 语义                           |
| ------------------- | ----------------------- | ------------------------------ |
| test  int1 -eq int2 | 测试整数是否相等 equals | equal 相等                     |
| test  int1 -ge int2 | 测试int1是否>=int2      | great than or equal 大于或等于 |
| test  int1 -gt int2 | 测试int1是否>int2       | greater than 大于              |
| test  int1 -le int2 | 测试int1是否<=int2      | less than or equal 小于或等于  |
| test  int1 -lt int2 | 测试int1是否<int2       | less than 小于                 |
| test  int1 -ne int2 | 测试整数是否不相等      | not equal 不等于               |

> [!NOTE]
>
> 前面表达式为`TRUE`, 则执行&&后面的第一条语句，否则执行第二条。



#### 2.3.3 文件测试

| 测试类型     | 结果                       | 语义      |
| ------------ | -------------------------- | --------- |
| test -d file | 文件是否目录               | directory |
| test –e file | 文件是否存在 exists        | exists    |
| test -f file | 文件是否常规文件           |           |
| test –L File | 文件存在并且是一个符号链接 | link      |
| test -r file | 指定文件是否可读           | read      |
| test -w file | 指定文件是否可写           | write     |
| test -x file | 指定文件是否可执行         |           |

#### 2.3.4 多重条件测试

```powershell
#语法
条件1 -a 条件二 && # 两个都成立，为真
条件1 -o 条件二 || # 只要有一个为真，则为真
! 条件 # 逻辑非，取反

num=520
[ -n $num -a $num -ge 520 ] && echo "marry you" || echo "go on"
```





## 3. 流程控制语句

### 3.1 if/else命令

```shell
if [ 条件 ]
	then
		程序
fi
#-----或者-----
if [ 条件 ];then
	程序
fi	

if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```

> [!NOTE]
>
> 1. `if`语句使用`fi`结尾。
> 2. then后面跟符号条件之后执行的程序，可以放在[]之后，用“;”分割，也可以换行写入，就不需要"；"了。 

### 3.2 多分支if条件语句

```shell
if condition
then
    command1 
    command2
    ...
    commandN
else
    command
fi

a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```

### 3.3 for循环

```shell
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done

for var in item1 item2 ... itemN; do command1; command2… done;

for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done

for str in "hello" "world" "shell";do
    echo $str
done  
```

### 3.4 while 语句

while循环用于不断执行一系列命令，也用于从输入文件中读取数据；命令通常为测试条件。其格式为： 

```shell
while condition
do
    command
done

#!/bin/bash
int=1
while(( $int<=5 ))
do
    echo $int
    let "int++"
done
```

### 3.5 until 循环

 until 循环执行一系列命令直至条件为 true 时停止。 

 until 循环与 while 循环在处理方式上刚好相反。 

 一般 while 循环优于 until 循环，但在某些时候—也只是极少数情况下，until 循环更加有用。 

```shell
until condition
do
    command
done

#!/bin/bash

a=0

until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

condition 一般为条件表达式，如果返回值为 false，则继续执行循环体内的语句，否则跳出循环。 

### 3.6 case

```shell
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac

echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```



### 3.7 跳出循环

 break和continue。 

#### break

break命令允许跳出所有循环（终止执行后面的所有循环）。

#### continue

continue命令与break命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。































