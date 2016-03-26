---
layout: post
title: linux软件安装
categories: unix
---

`apt-get`,`yum` ,`dnf`,`pacman`,`pip`,`brew`

#### ubuntu

    sudo apt-get install xxx //安装
    sudo apt-get remove xxx　//卸载

#### centos

    sudo yum install xxx
    sudo yum remove xxx

#### mac

mac的包管理工具默认是不带的，需要自己安装[homebrew](http://brew.sh/)，

homebrew安装的软件在／usr/local/bin,

`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

port安装的软件在[macports](https://www.macports.org/install.php),直接dmg安装就行，安装的软件在/opt/local/bin

	brew install xxx
	brew remove xxx
    port inst
    all xxx
    port remove xxx

#### 源码安装的

    ./configure //检查依赖关系是否满足
    make    //编译
    make install // 安装
    make　uninstall //卸载 如果软件源码删了，那么就不能了．
