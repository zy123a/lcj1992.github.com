---
layout: post
title: webservice入门
categories:
tags : webservice cxf
---

* [定义](#what)
  * [xml](#xml)
  * [wsdl](#wsl)
  * [uddi](#uddi)
  * [soap](#soap)
* [使用方式](#how-to-use)

### 定义 {#what}

![交互](/images/java_web/webservices.png)

##### XML {#xml}

web service 使用xml来传输数据，大多数语言都可以解析xml

##### Web Services Description Language {#wsdl}

[example](http://unibiz.ceair.com/UnifiedBusiness/services/mub2c_uni_refund_tkt_service?wsdl)

不同的系统之间交互需要遵循一定的规则:

* 一个系统应如何请求另一个系统的数据
* 请求中需要带哪些参数
* 产生的数据的格式（通常情况下，数据通过xml来进行传输，xml文件通过xsd来进行校验）
* 通讯失败了，应该展示怎样的错误信息

所有这些通过后缀名为`.wsdl`文件来定义。

##### Universal Description,Discovery and Intergration {#uddi}

一个叫做UDDI的目录定义了哪个软件系统应该连接那种数据类型，所以当一个软件系统需要传输特定的数据时，它会去uddi中查找哪个系统可以连接来接受这些数据。一旦它找到哪个系统应该被连接后，它会通过soap协议进行连接。服务提供者应该首先通过wsdl来校验数据，然后处理请求，并通过soap协议发送数据。

##### Simple Object Access Protocol {#soap}

soap 是web services交换格式化信息的协议。使用xml,依赖应用层协议（通常是http或者smtp）

### 使用 {#how-to-use}

JAX-WS are Java standard to build web service.

Apache CXF and Apache Axis 2 are two implementations of JAX-WS. They also offer JAX-RS implementations so that you can build Restful services.

cxf client:

    ./wsdl2java -d /Users/fool/xx/src/main/java -p com.xx.airlines.ceair.webservicenew.refund -client http://unibiz.xx.com/xx/mub2c_uni_refund_tkt_service?wsdl

axis client:

    java -Djava.ext.dirs=lib org.apache.axis.wsdl.WSDL2Java -p com.xx.airlines.mfair.webservice http://ebws.xx.com/etcwip/wsdl/CWIPService.wsdl

    <!-- java -classpath.:axis.jar:commons-logging-1.0.4.jar:commons-discovery-0.2.jar:saaj.jar:wsdl4j-1.5.1.jar:jaxrpc.jar:mail-1.4.5.jar \
    org.apache.axis.wsdl.WSDL2Java -p com.qunar.flight.flagship.provider.airlines.mfair.webservice \     
    http://ebws.travelsky.com/etcwip/services/CWIPService/wsdl/CWIPService.wsdl -->


*   http接口：air9 airchina csair kmair ..
*   webservice：
    *  cxf：ceair
    *  axis： hnair mfair scal shenzhenair spair ..

### 参考 {#ref}

[web service]<https://en.wikipedia.org/wiki/Web_service>

[soa]<https://en.wikipedia.org/wiki/Service-oriented_architecture>

[dubbo webservice]<http://dubbo.io/WebService+Protocol.htm>

[web services discovery]<https://en.wikipedia.org/wiki/Web_Services_Discovery>

[soap]<https://en.wikipedia.org/wiki/SOAP>

[Difference between Apache CXF and Axis]<http://stackoverflow.com/questions/1243247/difference-between-apache-cxf-and-axis>

[Difference between Jax-ws, axis2, cxf]<http://stackoverflow.com/questions/11566609/difference-between-jax-ws-axis2-cxf?lq=1>
