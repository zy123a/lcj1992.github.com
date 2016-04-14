---
layout: post
title : cookie和session
categories: java_web
tags: cookie session
---

### cookie

HTTP cookie ***由`服务端（产生方）`发送到`客户端（存储方）`，并存储在用户的客户端（浏览器）中．每次用户请求网站时，浏览器会将cookie发回给服务器．***cookie可以用来记录一些状态信息（如购物车中的商品）或者记录用户的浏览行为

#### 属性

|属性|说明|
|-|-|
|name|cookie名|
|value|cookie值|
|domain|哪个域名发送的，最长256字符|
|path|哪个路径发送的|
|secure|是否允许|
|httponly|是否仅能通过http和https渠道暴漏．防止客户端js执行，盗取cookie|
|max-age|持续时间|
|expires|过期时间,和上类似|

[cookie的实现](#java_cookie)

servlet-api 2.5之前版本（含）没有httpOnly这个字段，需要费点劲来设置httpOnly `cookie.setPath(isHttpOnly ? "/pathxx; HttpOnly;": "/pathxx");`

chrome中的cookie,***如果cookie不设置过期时间(`会话cookie`),表示这个cookie的生命周期为浏览器会话时期***，下图的过期时间那列Session就是这个意思
![cookie](/images/web/cookie.png)


### session
session需要使用cookie作为识别标志。HTTP协议是无状态的，session不能依据HTTP连接来判断是否为同一客户，因此服务器向客户端浏览器发送一个名为JSESSIONID的cookie，其值为该session的ID。该cookie为服务器自动生成的，***maxAge属性一般为-1，表示仅当前浏览器内有效，并且各浏览器窗口间不共享，关闭浏览器就会失效。***因此同一机器的两个浏览器窗口访问服务器时会生成两个不同的session

cookie的java实现
<a id="java_cookie"/>

    //
    // The value of the cookie itself.
    //

    private String name;	// NAME= ... "$Name" style is reserved
    private String value;	// value of NAME

    //
    // Attributes encoded in the header's cookie fields.
    //

    private String comment;	// ;Comment=VALUE ... describes cookie's use
				// ;Discard ... implied by maxAge < 0
    private String domain;	// ;Domain=VALUE ... domain that sees cookie
    private int maxAge = -1;	// ;Max-Age=VALUE ... cookies auto-expire
    private String path;	// ;Path=VALUE ... URLs that see the cookie
    private boolean secure;	// ;Secure ... e.g. use SSL
    private int version = 0;	// ;Version=1 ... means RFC 2109++ style
    private boolean isHttpOnly = false;


参考：

[1]<https://en.wikipedia.org/wiki/HTTP_cookie>

[2]<http://www.cnblogs.com/shiyangxt/archive/2008/10/07/1305506.html>