---
layout: post
title: maven依赖机制
categories: soft
tags: maven
---

#### 原汁原味
https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html

#### 依赖机制的介绍
依赖管理是maven一个著名的特性，也是maven优秀的一点。如果是一个单一的工程，管理依赖不是很难，

但是如果你开始处理一个包含有成十上百的模块的多模块的工程和应用时，这时候maven可以在高度的控制和稳定性方面给你提供较大的帮助

##### 传递依赖(transitive mediation)
传递依赖是maven2.0之后的一个特性，它允许你不用去发现和配置你的依赖的依赖，它自动把它include了。这个特性可以帮助你从你配置的远端仓库阅读项目文件(?)

一般情况下，所有的依赖，包含你工程使用的，同时包含这个工程继承自他的parents的。There is no limit to the number of levels that dependencies can be gathered from, and will only cause a problem if a cyclic dependency is discovered.

由于传递依赖，这个依赖关系图可能很大。由于这个原因，有一些额外的特性来决定那个dependencies来include进来。

##### 依赖调解(Dependency mediation)
这决定了当出现多版本时决定要选用那个版本。目前，maven2.0仅仅支持“nearest definition”，这意味着它将使用依赖关系树中离工程较近的版本，

你通过在工程pom里边显式的指定一个版本来保证一个版本。如果两个版本在依赖树中有同样的深度，直到2.0.8它不被定义谁将胜，但是自从2.0.9，按照声明的顺序，决定选择那个版本。

“nearest definition” 意味着使用依赖树中距离工程比较近的版本。例如，如果A依赖B依赖C依赖D2.0 并且A依赖E依赖D1.0,那么当编译 A时将会使用D1.0因为从A到D经过E更近。

你也可以显式的指定D2.0到A来强制使用D2.0

##### 依赖管理(Dependency management)
这允许项目的作者来直接指定要使用的artfacts的版本当他们出现在传递依赖中或者在依赖中没有指定版本。在本节的这个例子中，一个依赖直接被加到A上即使他没有被A直接使用。

A可以在他的dependencyManagement中包含D来直接控制要什么那个版本的D当被提及。

##### 依赖作用域(Dependency scope)
这允许你在适当的阶段加入依赖。

##### 排除依赖(Excluded dependencies)
如果工程依赖工程Y，工程Y依赖工程Z，工程X的所有者可以使用“exclusion”显式的排除项目

##### 选择依赖(Optional dependencies)
..

#### pom继承
子pom继承父pom<dependencyment>,<dependencies>等，但是自己子pom可重写。

父模块packing方式必须为pom才能被子模块继承,在父pom中modules节点中添加子模块，在子模块中添加parent
