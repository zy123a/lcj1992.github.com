---
layout: post
title: annotation-config与component-scan
categories: java_web
description: spring注解
keywords: annotation-config, component-scan
---


**注：好文章的搬运工！**
>   *`<context:annotation-config>`用来激活已经注册在application context中的beans的注解。（无论他是通过xml还是通过包扫描定义的）   
>   *`<context:component-scan>`可以做`<context:annotation-config>`做的，而且`<context:component-scan>`还可以扫包来发现和向application context中注册beans。  

## examples
类型为A，B，C的三个类，B，C 被注入到A中  

    package com.xxx;
    public class B {
      public B() {
        System.out.println("creating bean B: " + this);
      }
    }
    
    package com.xxx;
    public class C {
      public C() {
        System.out.println("creating bean C: " + this);
      }
    }
    
    package com.yyy;
    import com.xxx.B;
    import com.xxx.C;
    public class A { 
      private B bbb;
      private C ccc;
      public A() {
        System.out.println("creating bean A: " + this);
      }
      public void setBbb(B bbb) {
        System.out.println("setting A.bbb with " + bbb);
        this.bbb = bbb;
      }
      public void setCcc(C ccc) {
        System.out.println("setting A.ccc with " + ccc);
        this.ccc = ccc; 
      }
    }
配置文件如下：  

    <bean id="bBean" class="com.xxx.B" />
    <bean id="cBean" class="com.xxx.C" />
    <bean id="aBean" class="com.yyy.A">
      <property name="bbb" ref="bBean" />
      <property name="ccc" ref="cBean" />
    </bean>
当我们load context时，可以得到如下结果：

    creating bean B: com.xxx.B@c2ff5
    creating bean C: com.xxx.C@1e8a1f6
    creating bean A: com.yyy.A@1e152c5
    setting A.bbb with com.xxx.B@c2ff5
    setting A.ccc with com.xxx.C@1e8a1f6
这是老式风格的spring，让我们用注解来简化xml
首先，我们诸如BBB 和 CCC 在bean A中向这样：  

    package com.yyy;
    import org.springframework.beans.factory.annotation.Autowired;
    import com.xxx.B;
    import com.xxx.C;
    public class A { 
      private B bbb;
      private C ccc;
      public A() {
        System.out.println("creating bean A: " + this);
      }
      @Autowired
      public void setBbb(B bbb) {
        System.out.println("setting A.bbb with " + bbb);
        this.bbb = bbb;
      }
      @Autowired
      public void setCcc(C ccc) {
        System.out.println("setting A.ccc with " + ccc);
        this.ccc = ccc;
      }
    }
这样的话，我们就可以移除xml中的这两行：   

    <property name="bbb" ref="bBean" />
    <property name="ccc" ref="cBean" />
我的xml就可以简化为：  

    <bean id="bBean" class="com.xxx.B" />
    <bean id="cBean" class="com.xxx.C" />
    <bean id="aBean" class="com.yyy.A" />
当我们load context时，可以得到如下结果：   

    creating bean B: com.xxx.B@5e5a50
    creating bean C: com.xxx.C@54a328
    creating bean A: com.yyy.A@a3d4cf
ok，这不对啊！发生什么了？为什么我的属性没有被注入？
注解是一个很好的特性，但是他们自己不做任何   事情，他们仅仅是注解，你需要一个processing tool来发现这些注解，然后用他们来做些什么。`<context:annotation-config>`就是来干这个的，他找到和他定义在同一个application context的beans，并激活这些beans中的注解。
如果我更改我的xml如下：  

    <context:annotation-config />
    <bean id="bBean" class="com.xxx.B" />
    <bean id="cBean" class="com.xxx.C" />
    <bean id="aBean" class="com.yyy.A" />
当我load the application context 我得到了合适的结果：  

    creating bean B: com.xxx.B@15663a2
    creating bean C: com.xxx.C@cd5f8b
    creating bean A: com.yyy.A@157aa53
    setting A.bbb with com.xxx.B@15663a2
    setting A.ccc with com.xxx.C@cd5f8b
    
ok，当我移除了xml中的这两行然后添加了一行，结果很好。那我把xml中全删了，然后用注解来实现呢？  

    package com.xxx;
    import org.springframework.stereotype.Component;
    @Component
    public class B {
      public B() {
        System.out.println("creating bean B: " + this);
      }
    }
    
    package com.xxx;
    import org.springframework.stereotype.Component;
    @Component
    public class C {
      public C() {
        System.out.println("creating bean C: " + this);
      }
    }
    
    package com.yyy;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;
    import com.xxx.B;
    import com.xxx.C;
    @Component
    public class A { 
      private B bbb;
      private C ccc;
      public A() {
        System.out.println("creating bean A: " + this);
      }
      @Autowired
      public void setBbb(B bbb) {
        System.out.println("setting A.bbb with " + bbb);
        this.bbb = bbb;
      }
      @Autowired
      public void setCcc(C ccc) {
        System.out.println("setting A.ccc with " + ccc);
        this.ccc = ccc;
      }
    }
xml中仅仅保留：

    <context:annotation-config />
当重新load context时，nothing，没有beans被创建，没有beans被注入。  
这是因为，正如我第一段说的，`<context:annotation-config />`仅仅对已经注册在application context中的beans生效，当移除xml中的配置时，没有beans被创建，`<context:annotation-config />`没有作用的对象。  
但是这对于`<context:component-scan>`不是问题，他可以扫包来创建beans，然后注入。  
然后我们更改xml配置如下：  
    
    <context:component-scan base-package="com.xxx" />
当load context时，得到如下输出：  
    
    creating bean B: com.xxx.B@1be0f0a
    creating bean C: com.xxx.C@80d1ff
还是少了点啥！为什么呢？
原来A在com.yyy包中，而我xml中只用了包com.xxx,所以会遗漏A，更改xml配置如下：  

    <context:component-scan base-package="com.xxx,com.yyy" />    
得到期望的结果.
保持A,B,C的注解，修改xml配置如下：  
    
    <context:component-scan     base-package="com.xxx" />
    <bean id="aBean" class="com.yyy.A" />
我们仍然得到正确的结果：  
    
    creating bean B: com.xxx.B@157aa53
    creating bean C: com.xxx.C@ec4a87
    creating bean A: com.yyy.A@1d64c37
    setting A.bbb with com.xxx.B@157aa53
    setting A.ccc with com.xxx.C@ec4a87  
即使A类的bean没有被扫描到，`<context:component-scan>`仍然能够对application context中的所有beans起作用，即使A是通过xml注册的。  
    但是如果我们修改我们的xml配置如下，会不会得到重复的beans呢？  
    
    <context:annotation-config />
    <context:component-scan base-package="com.xxx" />
    <bean id="aBean" class="com.yyy.A" />  
没有重复，我们仍然得到了期望的结果：  
    
    creating bean B: com.xxx.B@157aa53
    creating bean C: com.xxx.C@ec4a87
    creating bean A: com.yyy.A@1d64c37
    setting A.bbb with com.xxx.B@157aa53
    setting A.ccc with com.xxx.C@ec4a87  
这是因为如果有`<context:component-scan>`了，`<context:annotation-config />`会被忽略，spring只运行他们一次。  
    即使你注册了多个processing tools，Spring仍然会确定你只做了一次，比如你的xml如下：  
    
    <context:annotation-config />
    <context:component-scan base-package="com.xxx" />
    <bean id="aBean" class="com.yyy.A" />
    <bean id="bla" class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor" />
    <bean id="bla1" class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor" />
    <bean id="bla2" class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor" />
    <bean id="bla3" class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor" />
仍然会得到一样的结果：  
    
    creating bean B: com.xxx.B@157aa53
    creating bean C: com.xxx.C@ec4a87
    creating bean A: com.yyy.A@25d2b2
    setting A.bbb with com.xxx.B@157aa53
    setting A.ccc with com.xxx.C@ec4a87
通过学习，你会发现`<context:component-scan/>`能够识别<`context:annotation-config/>`的超集，也就是
>   *`@Component`, `@Service`, `@Repository`, `@Controller`, `@Endpoint`       
>   *`@Configuration`, `@Bean`, `@Lazy`, `@Scope`, `@Order`, `@Primary`, `@Profile`, `@DependsOn`, `@Import`, `@ImportResource`
      
As you can see `<context:component-scan/>` logically extends `<context:annotation-config/>` with CLASSPATH component scanning and Java @Configuration features.


[1]<http://stackoverflow.com/questions/7414794/difference-between-contextannotation-config-vs-contextcomponent-scan>