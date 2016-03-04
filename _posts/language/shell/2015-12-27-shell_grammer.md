---
layout: post
title: shell 控制语句
categories: language
tags: shell
---

#### if
测试命令:

test or [ ﻿判定语句 ]

***判定语句 和方括号([ ])之间必须有空格，否则会有语法错误***

1.Integer ( Number without decimal point)
2.File types
3.Character strings

#### 数字

| shell脚本中的比较运算符	 |含义	|test	|[ expr ]|
|-|-|-|-|
|-eq|	等于|if test 5 -eq 6 |if [ 5 -eq 6 ]
|-ne	|不等于|if test 5 -ne 6|if [ 5 -ne 6 ]
|-lt	|小于|if test 5 -lt 6|if [ 5 -lt 6 ]
|-le|	小于等于|if test 5 -le 6|if [ 5 -le 6 ]
|-gt	|大于|if test 5 -gt 6|if [ 5 -gt 6 ]
|-ge	|大于等于|if test 5 -ge 6|if [ 5 -ge 6 ]

#### 字符串

|Operator	|Meaning|
|-|-|
|string1 = string2	|string1 is equal to string2
|string1 != string2	|string1 is NOT equal to string2
|string1	|string1 is NOT NULL or not defined
|-n string1	|string1 is NOT NULL and does exist
|-z string1	|string1 is NULL and does exist

#### 文件

|Test	|Meaning|
|-|-|
|-s file  	|Non empty file
|-f file  	|Is File exist or normal file and not a directory
|-d dir   	|｜Is Directory exist and not a file
|-w file 	|Is writeable file
|-r file  	|Is read-only file
|-x file  	|Is file is executable

#### 逻辑运算符

|Operator           	｜Meaning
|-|-|
|! expression	|Logical NOT
|expression1  -a  expression2	|Logical AND
|expression1  -o  expression2	|Logical OR

#### if

    //if
    if [ xxxexpression ]
    then
       xxxCommand
    fi

    //if else
    if [ expression ]
    then
       Statement(s) to be executed if expression is true
    else
       Statement(s) to be executed if expression is not true
    fi

    //if elif else
    if [ expression 1 ]
    then
       Statement(s) to be executed if expression 1 is true
    elif [ expression 2 ]
    then
       Statement(s) to be executed if expression 2 is true
    else
       Statement(s) to be executed if no expression is true
    fi
#### case
取值后面必须为关键字 in，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;。;; 与其他语言中的 break 类似，意思是跳到整个 case 语句的最后.取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。

    case 值 in
    模式1)
        command1
        ...
        ;;
    模式2）
        command1
        ...    ;;...
    *)
        command1
        ...
        ;;
    esac

#### for
列表是一组值（数字、字符串等）组成的序列，每个值通过空格分隔。每循环一次，就将列表中的下一个值赋给变量。

in 列表是可选的，如果不用它，for 循环使用命令行的位置参数。

    for 变量 in 列表
    do
        command1
        command2
        ...
        commandN
    done

或者这样

    for (( expr1; expr2; expr3 ))
    do
       .....
       .....   (repeat all statements between do and
       done until expr2 is TRUE)Done

#### while
条件判定语句

    while command
    do
       Statement(s) to be executed if command is true
    done

#### until
跟c中while和do-while的区别

    until command
    do
       Statement(s) to be executed until command is true
    done

#### 终止语句
1.break 跳出所有循环，这和c，java中的break不一样

2.continue 跳出当前循环

3.break,continue后都可跟一个数字，表明跳出第几层循环

#### 函数
Shell 函数的定义格式如下：

    function_name () {
        list of commands
        [ return value ]
    }

如果你愿意，也可以在函数名前加上关键字 function：

    function function_name () {
        list of commands
        [ return value ]
    }

函数返回值，可以显式增加return语句；如果不加，会将最后一条命令运行结果作为返回值。

调用函数时不加括号

debug

-v Print shell input lines as they are read.
-x After expanding each simple-command, bash displays the expanded value of PS4 system variable, followed by the command and its expanded arguments.

函数传参例子

    #!/bin/bash
    name=$1
    age=$2
    sayHello(){
       echo "I am $name at age of $age"
       return 0
    }
    sayHello $name $age

|特殊符号 |含义|
|-|-|
|$#|	传递给函数的参数个数。|
|$*	|显示所有传递给函数的参数。|
|$@	|与$*相同，但是略有区别，请查看Shell特殊变量。|
|$?|	函数的返回值。|