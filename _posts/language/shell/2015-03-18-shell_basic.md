---
layout: post
title: shell基础
categories: language
tags: shell
---


#### 变量

1.`name=value`   或者声明一个未赋值的变量`delcare name`

**注意，变量名和等号之间不能有空格，这可能和你熟悉的所有编程语言都不一样。**

命名规则跟c一样，开头是字母或者下划线，然后字母下划线或者数字，大小写敏感

2.引用变量 `$name`

3.使用`readonly`命令可以将变量定义为只读变量，只读变量的值不能被改变。

4.使用 unset 命令可以删除变量。变量被删除后不能再次使用；unset 命令不能删除只读变量。

5.变量类型：局部变量，环境变量，shell变量

#### 特殊变量

|变量|含义|
|-|-|
|$0|	脚本名字
|$n	|输入参数
|$#	|传递给脚本或函数的参数个数
|$*	|传递给脚本或函数的所有参数
|$@	|传递给脚本或函数的所有参数
|$?	|上个命令的推出状态或函数的返回值
|$$	|当前shell进程id，对于shell脚本，就是这些脚本所在进程的id

$* 和 $@ 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。

但是当它们被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。

#### 引号

**单引号 '**  全当字符串对待

**双引号 "**  除了\ $ 当字符串对待

**back quote `**  命令,在shell 中 $() 与 ``等效。

#### 读

read variable

#### 通配符

\* 匹配所有字符

?匹配单一字符

[...] 匹配任一字符

#### 多命令执行

command1;command2

#### shell替换

[详见](/2016/01/17/args_replace)

#### 文件重定向

\> 覆盖

\>\> 追加

\< 将文件重定向到命令

标准输入 0

标准输出 1

标准错误 2

黑洞 /dev/null

2>xx.log **2和\>之间不能有空白字符**

通过使用 <(some command) 可以将输出视为文件。例如，对比本地文件 /etc/hosts 和一个远程文件：

      diff /etc/hosts <(ssh somehost cat /etc/hosts)

#### 条件执行(and or)

`command1 && command2` 当且仅当命令1返回0,命令2才执行

`command1 || command2` 当且仅当命令1返回非0,命令2才执行

#### shell 浮点数

expr支持加减乘除，但是不支持浮点数，可以借助awk来实现。

awk 'BEGIN {print $var1 / $var 2}'


#### shell数组

sites=(kmair air9 spair)

遍历数组

`for i in ${sites[*]}`

#### 参考

[1]<http://www.freeos.com/guides/lsst/>

[2]<http://c.biancheng.net/cpp/view/2736.html>

[3]<http://www.ibm.com/developerworks/cn/linux/l-cn-shell-debug/>