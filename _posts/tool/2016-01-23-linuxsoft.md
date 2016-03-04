---
layout: post
title: linux的软件安装卸载
categories: tool
---

`apt-get`,`yum` ,`dnf`,`pacman`,`pip`,`brew`

#### ubuntu

    sudo apt-get install xxx //安装
    sudo apt-get remove xxx　//卸载

#### centos

    sudo yum install xxx
    sudo yum remove xxx

#### 源码安装的

    ./configure //检查依赖关系是否满足
    make    //编译
    make install // 安装
    make　uninstall //卸载 如果软件源码删了，那么就不能了．
