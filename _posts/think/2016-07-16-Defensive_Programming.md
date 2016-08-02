---
layout: post
title: 防御式编程
categories: think
---
todo

1.  enum和字面量
2.  范型
3.  框架该做的事
4.  接口幂等

作为一个技术负责人的你，你当然有权利通过使用非技术的手段来达到你的目的。比如：你在团队内部明文规定，“XX类只能有一个全局实例，如果某人使用两次以上，那么该人将被处于2000元的罚款！”（呵呵），你当然有权这么做。但是如果你的设计的是东西是一个类库，或是一个需要提供给用户使用的API，恐怕你的这项规定将会失效。因为，你无权要求别人会那么做。所以，这就是为什么，我们希望通过使用技术的手段来达成这样一个目的的原因。

[深入浅出单实例Singleton设计模式](http://coolshell.cn/articles/265.html)


防御式编程，不要相信任何人和服务。别相信你的下游说，我就调你三次，你放心吧，没事的。别信，肯定有鬼，你要做好对自身的保护，也不要相信下游说别人的提供的服务放心地使，哥向你保证五个9的可靠性，没有一个服务能做到100%的可靠的，这是必然的，即使是5个9，也有损失的时候，别相信他，要做好对下游的依赖和熔断；

[美团外卖系统架构演进与稳定性的探索]<http://mp.weixin.qq.com/s?src=3&timestamp=1470106502&ver=1&signature=wz0C0rOE4hyEHQTWtOplUTyZH1CHKoIewuP9cgQC2EFTTcjstLiFn5Omukel2VOFA65KhMNcEHmI7kLmC0WU*wyWialxdDSJFyzW1AYjAaRN4E82opMvOkyjTfCeIRqpUASQynAc4q*Lw3SrQ-dLtdohI-eBfP1R7HHr3hpqrY4=>
