---
layout: post
title: airlineAv调优记
categories: soft
tags: tomcat source
---

airlineAv每隔几周就得重启一次,从前人的前人,再到前人,听说经常这样,而且还修了很多次.

从16年开始多家航司都不合作,只剩下几家小的,量小问题没暴漏,从东航4月6号重新上线,然后就隔个一周左右就跪个一次,所以问题很可能在东航.于是在机器再次跪掉之后,决定搞一下.

ps: 航司接口不一样,有的是http,有的是webservice(axis和cxf的也都有)

![airlineAv tcp连接监控](/images/soft/tcp_established.png)

现象是: tomcat的connector配了2000个线程,还老是超.目前就几家航司在线,qps30左右,五台机器,平均每台也就10啊,这连接最高还2000+,好不正常.

1.先是看了下gc的状况,old区接近100%,堆的设置-Xms2048m -Xmx2048m,要知道这是8g的机器,所以我改了下jvm配置Xms,Xmx都改成了4g.

***不管有用没用,有资源不用浪费, 4g + 2000 * 1m(linux hotspot默认栈大小) = 6g***

2.tomcat也没死,就是接收不了请求了,healthcheck.html也请求不了.因为tomcat的connector打满了,这点可以理解.

基本可以断定是webservice的没设超时时间,或者超时时间没生效,导致坏连接一直没释放,不断堆积.

webservice又没搞过,所以想通过不改代码,不走发布,直接改tomcat配置,设置tomcat的超时时间来达到目的.

找了下tomcat的文档,好像没有一个设置读超时的配置(倒是让我想看一下[tomcat的源码](/2016/04/26/tomcat_source)也是好事)

冰神按照国际旗舰店的超时时间设置,观察了几天还是没生效,最后勇哥道出了其中的要诀,webservice生成客户端的方法不一样,axis与cxf,东航采用的cxf的方式生成webservice client的,而超时时间设置是采用axis的.

所以又google下,cxf的超时时间设置,写了个demo来测试下.[demo地址](https://github.com/lcj1992/learn/blob/master/java/src/main/java/cxf),超时时间设置见CxfClientUtil.java

然后直接把util拷进airlineav,引入jar包就提测了,qa测试时候又报找不着方法.没有引到正确地包,发现对应的类在axis-wsdl4j 和wsdl4j都含有,查了下两个包的历史渊源,干掉了axis-wsdl4j,运行时的method找不到也fix了,发不上线.

截止目前,发布一周(4/29 - 5/6)tcp estalished的维持在30左右了,再也不是2k+了(如上图).

[四种soap引擎vs]<http://blog.sina.com.cn/s/blog_a9fbb4dc01014be5.html>

[cxf conflict]<http://stackoverflow.com/questions/6066054/whats-wrong-with-my-apache-cxf-client>

[axis-wsdl4j and wsdl4j]<http://stackoverflow.com/questions/8219215/difference-between-axis-wsdl4j-and-wsdl4j>

[jvm线程栈默认大小]<https://www.zhihu.com/question/27844575>

[cxf client超时时间设置]<http://blog.csdn.net/wqmain/article/details/8647416>

[tomcat8官方文档]<https://tomcat.apache.org/tomcat-8.0-doc/config/service.html>