---
layout: post
title: 几个不错的软件(mac&llinux)
categories: tool
tags: alfred oh-my-zsh zsh z switchHosts
---

*   [zsh](#zsh)
*	[theFuck](#thefuck)
*	[z](#z)
*	[参考](#ref)
    
### zsh-终极shell {#zsh}

***1.安装zsh***

*   `cat /etc/shells`你会发现mac是有自带zsh的,但是和linux一样，默认的终端都是bash(ubuntu安装`sudo apt-get install zsh`) 
*   安装完成之后设置当前用户使用zsh: `chsh -s /bin/zsh`。但因为其配置复杂，所以很多人无人问津，
*   然后有人开发了个让你快速上手zsh的项目叫做［oh－my－zsh］

***2.安装oh－my－zsh***

[github地址](https://github.com/robbyrussell/oh-my-zsh)

安装：`sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`

1.	quick-look filename 可以直接预览文件，man-preview grep 可以生成 grep手册 的pdf 版本等
2.	强大的历史纪录功能，输入 grep 然后用上下箭头可以翻阅你执行的所有 grep 命令
3.	各种补全：路径补全、命令补全，命令参数补全，插件内容补全等等。触发补全只需要按一下或两下 tab 键，补全项可以使用 ctrl+n/p/f/b上下左右切换。比如你想杀掉 java 的进程，只需要输入 kill java + tab键，如果只有一个 java 进程，zsh 会自动替换为进程的 pid，如果有多个则会出现选择项供你选择。ssh + 空格 + 两个tab键，zsh会列出所有访问过的主机和用户名进行补全

***3.使用zsh***

1.	安装z插件，只需在～／.zshrc的plugin中添加`plugins=(git xx z)`,然后你就可以z xx目录之间随意跳转了.如果你没有使用oh-my-zsh框架，也可以使用z命令，下载z.sh,[git地址](https://github.com/rupa/z/),然后在环境变量中指定z.sh的路径`. /home/lcj/tool/z.sh`

### theFuck

纠正命令
[github地址](https://github.com/nvbn/thefuck)

### z

目录跳转,真的是神器

[github地址](https://github.com/rupa/z)

## 参考 {#ref}
 
[1]<http://macshuo.com/?p=676>

[2]<http://www.cnblogs.com/chijianqiang/p/alfred.html>

[3]<http://zeroturnaround.com/rebellabs/5-unexpectedly-useful-command-line-tools-you-might-overlook/>

[4]<http://www.zhihu.com/question/28182203>