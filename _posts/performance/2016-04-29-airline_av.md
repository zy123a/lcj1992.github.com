---
layout: post
title: airlineAv调优记
categories: work
tags: webservice airline av
---

### 背景

1.  五台机器,十几家航司

2.  上游对接抓取系统(离线用于旗舰店报价展示),下游对接航司av接口

3.  每隔几周就得重启一次,从前人的前人,再到前人,听说经常这样,而且还修了很多次.

4.  航司接口不一样,有的提供http,有的提供webservice(axis,cxf)
  
5.  年后一直保持稳定(多家航司不合作了),在东航4月6号重新上线后,就保持一周左右跪一次的节奏(量大了),于是决定搞一下.

6.  tcp established高达2k:

![airlineAv tcp连接监控](/images/performance/airlineav_tcp_established.png)

### 现象与处理

在第一次跪掉之后

1.  dump了线程堆栈,大量东航相关runnable的线程,tomcat connector的线程用到了快2000,看了下机器的监控确实tcp连接ESTABLISHED的近2000了(不一般都是Close_wait,time_wait多么?),tomcat还活着,但是接收不了请求了.

2.  先看了下gc的情况,old区吃满了,Xms、Xmx都是配的2g(五台机器都是8g的哎),于是我把它们都改成了4g,把tomcat的连接数改成2500了(可是单机正常的qps就15-25啊).
4g + 2500 * 1m(linux64位 hotspot默认栈大小) = 6.5g***,然后又重启了.

依旧在一周左右时间,连接再次占满,依旧是东航,扒代码,冰神按照国际旗舰店的webservice的超时时间设置方式加了超时时间,问题依旧在一周后出现.

然后debug时,发现webservice似乎也设置timeout,最后勇哥道出了其中的缘由:东航的的webservice client是通过cxf生成的,而国际旗舰店的是按照axis的方式的.

于是乎,google之: `cxf webservice 超时时间`(没搞过webservice啊),照着网上的搞了个[demo](https://github.com/lcj1992/learn/blob/master/java/src/main/java/cxf),超时时间设置见CxfClientUtil.java

然后引入新的jar包,又报method找不到,包冲突maven选错了jar.对应的类在axis-wsdl4j.jar和wsdl4j.jar,查了下两个包的历史渊源,干掉了axis-wsdl4j,fix了,发布上线.

截止目前,发布一周(4/29 - 5/6)tcp estalished的维持在30左右了,再也不是2k+了(如上图),内存used的也从3g变到了5g.

![airlineav 内存](/images/performance/airlineav_mem.png)

[四种soap引擎vs]<http://blog.sina.com.cn/s/blog_a9fbb4dc01014be5.html>

[cxf conflict]<http://stackoverflow.com/questions/6066054/whats-wrong-with-my-apache-cxf-client>

[axis-wsdl4j and wsdl4j]<http://stackoverflow.com/questions/8219215/difference-between-axis-wsdl4j-and-wsdl4j>

[jvm线程栈默认大小]<https://www.zhihu.com/question/27844575>

[cxf client超时时间设置]<http://blog.csdn.net/wqmain/article/details/8647416>

[tomcat8官方文档]<https://tomcat.apache.org/tomcat-8.0-doc/config/service.html>