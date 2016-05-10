---
layout: post 
title: 通过tcpdump学习网络协议
categories: network
tag: tcpdump tcp 
---


dump下线上一个包[tcpdump入门](/2015/03/20/network#tcpdump)

[chuangjian.li@l-xxxx.f.cn1 ~]$ sudo tcpdump -nnvvX host  xxx.xx.xxx.xx -c 3
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
20:17:08.896034 IP (tos 0x28, ttl 49, id 16341, offset 0, flags [none], proto TCP (6), length 1400)
    218.80.232.44.80 > 10.86.212.176.33314: Flags [.], cksum 0x195c (correct), seq 1024948164:1024949512, ack 2128146148, win 65535, options [nop,nop,TS val 2979322617 ecr 968275531], length 1348
	0x0000:  4528 0578 3fd5 0000 3106 a2ff da50 e82c  E(.x?...1....P.,
	0x0010:  0a56 d4b0 0050 8222 3d17 77c4 7ed8 eee4  .V...P."=.w.~...
	0x0020:  8010 ffff 195c 0000 0101 080a b194 daf9  .....\..........
	0x0030:  39b6 b64b 4854 5450 2f31 2e31 2032 3030  9..KHTTP/1.1.200
	0x0040:  204f 4b0d 0a44 6174 653a 204d 6f6e 2c20  .OK..Date:.Mon,.
	0x0050:  3039 204d 6179 2032 3031 3620 3132 3a31  09.May.2016.12:1
	0x0060:  343a 3339 2047 4d54 0d0a 5365 7276 6572  4:39.GMT..Server
	0x0070:  3a20 4170 6163 6865 0d0a 582d 4672 616d  :.Apache..X-Fram
	0x0080:  652d 4f70 7469 6f6e 733a 2053 414d 454f  e-Options:.SAMEO
	0x0090:  5249 4749 4e0d 0a56 6172 793a 2041 6363  RIGIN..Vary:.Acc
	0x00a0:  6570 742d 456e 636f 6469 6e67 0d0a 5472  ept-Encoding..Tr
	0x00b0:  616e 7366 6572 2d45 6e63 6f64 696e 673a  ansfer-Encoding:
	0x00c0:  2063 6875 6e6b 6564 0d0a 436f 6e74 656e  .chunked..Conten
	0x00d0:  742d 5479 7065 3a20 7465 7874 2f78 6d6c  t-Type:.text/xml
	0x00e0:  3b63 6861 7273 6574 3d55 5446 2d38 0d0a  ;charset=UTF-8..
	0x00f0:  436f 6e6e 6563 7469 6f6e 3a20 4b65 6570  Connection:.Keep
	0x0100:  2d41 6c69 7665 0d0a 0d0a 3166 6638 0d0a  -Alive....1ff8..

然后这篇文章就是说下上述这些数据是怎么和下边的ascii码对应上的(ip和tcp)

#### 基础 {#basic}

在网络iso的七层模型中,TCP位于第四层-Transport(传输)层,IP在第三层—Network层，ARP在第二层—Data Link层，
在第二层上的数据，我们叫Frame，在第三层上的数据叫Packet，第四层的数据叫Segment

##### ip packet structure

![ip header](/images/network/ip_header.png)

1.  Version:前四个字节为版本号: 0100  IPv4   0110 IPv6
2.  IHL: Internet Header Length,四个字节, 它是32bits字的数量(32 * IHL),最小为5(0101),最大为15(1111),所以ip头的长度\[5*32=160bits=20bytes,15*32=480bits=60bytes\]
3.  
4.  
##### tcp h
![tcp header](/images/network/tcp_header.png)


#### 参考 {#ref}

[wiki百科ip]<https://en.wikipedia.org/wiki/IPv4#Header>

[wiki百科tcp]<https://en.wikipedia.org/wiki/Transmission_Control_Protocol>

[tcp的那些事儿[上]]<http://coolshell.cn/articles/11564.html>

[tcp的那些事儿[下]]<http://coolshell.cn/articles/11609.html>

