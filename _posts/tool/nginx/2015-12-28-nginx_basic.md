---
layout: post
title: nginx简单使用
categories: tool
tags: nginx
---

一个人访问http://hehe.foolchild.com/，浏览器默认是80（listen）端口，然后会根据
localtion的映射关系访问 http://fuck/，再127.0.0.1:8080或者10.86.41.3：8080替换其中的fuck
（server_name）+ （listen）+（location）是用户访问的网址，
经（upstream）处理后的（proxy_pass）是真实的服务器上的地址。

     http {
            upstream fuck
    
        { 
              server 127.0.0.1:8080; 
              server 10.86.41.3:8080 
            }
        
            server {
                listen       80;
                server_name  pricelock.foolchild.com;
        
                location /
           {                 
                proxy_pass  http://fuck/;             
                }
            }  
        }

公司的源里的nginx自带lua模块,直接安装就可以,但是可能会存在版本问题,

1.  更新软件源
2.  查看nginx依赖的动态链接库 ldd \`which nginx\`

参考

[1]<http://nginx.org/en/docs/>

[2]<http://www.cnblogs.com/xiaogangqq123/archive/2011/03/02/1969006.html>