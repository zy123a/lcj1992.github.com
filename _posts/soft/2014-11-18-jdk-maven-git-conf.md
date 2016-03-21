---
layout: post
title: jdk maven git 配置
categories: soft
tags: soft jdk maven git
---

### jdk
*	`JAVA_HOME` : &nbsp; &nbsp;		**D:\q\java\default**   mac: \`/usr/libexec/java_home\`
	(**你的jdk的安装目录，这里创建了一个软连接，指向了jdk的安装目录**)  
*	`path`:  &nbsp;&nbsp;			**;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin**(**unix下是PATH**,之间已`:`分隔)
*	`CLASSPATH`: 	&nbsp;&nbsp;	**.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar**（**前面的.一定要加**）

### maven

*	`MAVEN_HOME`:	&nbsp;&nbsp;	**D:\bin\apache-maven**
*	`path`:		&nbsp;&nbsp;		**;%MAVEN_HOME%\bin**    
在.m2文件夹下注意要编辑settings.xml文件  
windows下是在C:\Users\xxx.li下
linux是在家目录下。
***mac 如果是brew 安装的软件,默认会在/usr/local/bin下, 在PATH中,所以不需要配置maven的环境变量***

### git
*	`GIT_HOME`:		&nbsp;&nbsp;	**D:\Program Files (x86)\Git**
*	`path`:		&nbsp;&nbsp;		**;%GIT_HOME%\cmd;**

### idea配置

file->settings
