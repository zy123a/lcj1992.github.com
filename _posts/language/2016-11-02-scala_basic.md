---
layout: post
title: scala基础
categories: language
tags: scala
---

特性：
1. 面向对象
2. 函数式
3. 静态类型
4. 运行在jvm上
5. 可以执行java代码
6. 善于处理并发

scala  vs  java

1. 所有的类型都是对象
2. 类型推断
3. 嵌套函数
4. 函数也是对象
5. 领域特定语言？？
6. traits
7. closures 闭包
8. 并发支持灵感来自Erlang


scala 编程：一堆对象，通过调用彼此的方法来进行通信。

Object
class
Methods
Fields
Closure
Traits 特征 类似于java8的函数式接口



基本语法：
保留关键字：


数据类型：
Unit 对应于空值
Null 空引用
Nothing  任何类型的子类型
Any 任何类型的超类
AnyRef 任何引用类型的超类


变量声明：

var  可变变量
val  不可变变量

        val or val VariableName : DataType = [Initial Value]

1. 声明变量时，可以不指定数据类型，可以不初始化
2. 支持multiple  assignments

         val (myVar1 : Int,myVar2: String) = Pair(40,"hello")



extents

内部类

方法声明：

        def functionName ([list of parameters]) : [return type]    


https://www.tutorialspoint.com/scala/
