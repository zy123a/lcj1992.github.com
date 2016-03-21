---
layout: post
title: 使用mac中几个不错的软件
categories: unix
tags: alfred oh-my-zsh zsh z switchHosts
---
*   [alfred](#alfred)
*   [zsh](#zsh)
*   [switchHosts](#switchhosts)


### alfred

*   快捷键 opt + space
*   打开应用程序
*   查找文件 回车定位文件 cmd + 回车 定位文件所在文件夹
*   **通过find、open、in等关键词搜索文件,find是定位文件，open是定位并打开文件，in是在文件中进行全文检索**
*   计算器
*   输入>即可直接运行shell命令 (收费)
    
### zsh-终极shell {#zsh}

***1.安装zsh***

*   `cat /etc/shells`你会发现mac是有自带zsh的,但是和linux一样，默认的终端都是bash(ubuntu安装`sudo apt-get install zsh`) 
*   安装完成之后设置当前用户使用zsh: `chsh -s /bin/zsh`。但因为其配置复杂，所以很多人无人问津，
*   然后有人开发了个让你快速上手zsh的项目叫做［oh－my－zsh］

***2.安装oh－my－zsh***

[github地址](https://github.com/robbyrussell/oh-my-zsh)

安装：`sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`

***3.使用zsh***

1.	安装z插件，只需在～／.zshrc的plugin中添加`plugins=(git xx z)`,然后你就可以z xx目录之间随意跳转了.如果你没有使用oh-my-zsh框架，也可以使用z命令，下载z.sh,[git地址](https://github.com/rupa/z/),然后在环境变量中指定z.sh的路径`. /home/lcj/tool/z.sh`

### switchHosts

windows和mac都有,测试切换hosts，之前是手工改/etc/hosts的，后来感觉还是慢。[github地址](https://github.com/oldj/SwitchHosts)


### theFuck

[github地址](https://github.com/nvbn/thefuck)


## 参考 {#ref}
 
[1]<http://macshuo.com/?p=676>

[2]<http://www.cnblogs.com/chijianqiang/p/alfred.html>

[3]<http://zeroturnaround.com/rebellabs/5-unexpectedly-useful-command-line-tools-you-might-overlook/>

[4]<http://www.zhihu.com/question/28182203>
