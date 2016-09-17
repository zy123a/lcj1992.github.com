---
layout: post
title: spring常用配置
categories: java_web
tags: spring
---

### 会看xsd
spring的配置xsd都有详细说明，然后稍微懂点英语，会看xsd就ok了。
[会看xsd](/2015/12/27/xsd)

下是一些常用配置，讲究实用

#### 读取配置文件

    1.<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath*:*.properties</value>
                <value>classpath*:*/*.properties</value>
            </list>
        </property>
    </bean>
    ----------
    2.<context:property-placeholder location="classpath*:*.properties" file-encoding="UTF-8"/>

#### 依赖注入

    1.<context:component-scan base-package="com.xxx">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    ----------
    2.<context:annotation-config/>
    <!--前者除了依赖注入外还可以实例化bean,并将其注入进spring容器中，后者则只对已经在spring容器中的bean有效-->

#### 使用aspectj aop来初始化上下文

    <!-- 使用aop来初始化上下文 -->
    <aop:aspectj-autoproxy proxy-target-class="true"/>
        <bean id="AopForContext" class="com.xxx.util.AopForContext">
    </bean>

#### 开启事务

    <tx:annotation-driven transaction-manager="transactionManager" mode="aspectj" />

[spring事务的两种方式](/2015/12/28/spring_transaction)

#### 开启spring mvc controller

1.  如果没有@ResponseBody的话，spring mvc的返回走的是spring的视图渲染器，可以设置model，可以forword://xxxUrl?orderNo
2.  如果有ResponseBody的话，返回return的结果，可以设置message-Converter处理返回值例如spring自带的MappingJackson2HttpMessageConverter


        <!-- Enables the Spring MVC @Controller programming model -->
        <mvc:annotation-driven>
            <mvc:message-converters>	<!--支持的media类型s-->
                <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                    <property name="supportedMediaTypes">
                        <list>
                            <value>text/plain;charset=UTF-8</value>
                            <value>text/xml;charset=UTF-8</value>
                            <value>text/html;charset=UTF-8</value>
                        </list>
                    </property>
                </bean>	<!--json格式的序列化与反序列化-->
                <bean id="mappingJacksonHttpMessageConverter"
                      class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
                    <property name="supportedMediaTypes">
                        <list>
                            <value>application/json;charset=UTF-8</value>
                        </list>
                    </property>
                    <property name="objectMapper">
                        <bean class="org.codehaus.jackson.map.ObjectMapper">
                            <property name="serializationInclusion" value="NON_NULL"/>
                        </bean>
                    </property>
                </bean>
            </mvc:message-converters>
        </mvc:annotation-driven>

#### <mvc:default-servlet-handler />
项目根目录下的index.jsp不生效，index.jsp是纯文本输出

解决方案与说明：在controller的配置文件中添加如下，告诉spring mvc将静态资源的处理交回Web应用服务器tomcat处理。(spring mvc主要是使用restful风格的)

#### pathVariable

    @RequestMapping(value="/comment/{blogId}", method=RequestMethod.POST)
    public void comment(@PathVariable int blogId, HttpSession session) {
    }

在该例子中，blogId是被@PathVariable标记为请求路径变量的，如果请求的是/blog/comment/1.do的时候就表示blogId的值为1

同样@RequestParam也是用来给参数传值的，但是它是从头request的参数里面取值，相当于request.getParameter("参数名")方法。
它的取值规则跟@PathVariable是一样的，当没有指定的时候，默认是从request中取名称跟后面接的变量名同名的参数值，
当要明确从request中取一个参数的时候使用@RequestParam("参数名")

#### @Scope("prototype")

    @Component("QueryMessageTask")
    @Scope("prototype")
    public class QueryMessageTask  implements Callable<List<TaskInfo>>
    Scope默认是singleton(单例).

#### map属性

    <!--邮件模板-->
    <bean id="velocityEngine"
          class="org.springframework.ui.velocity.VelocityEngineFactoryBean">
        <property name="resourceLoaderPath">
            <value>classpath:</value>
        </property>
        <property name="velocityProperties">
            <props>
                <prop key="input.encoding">UTF-8</prop>
                <prop key="output.encoding">UTF-8</prop>
                <prop key="contentType">text/html;charset=UTF-8</prop>
                <prop key="resource.loader">class</prop>
                <prop key="class.resource.loader.class">
                    org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader
                </prop>
                <prop key="file.resource.loader.cache">false</prop>
                <prop key="file.resource.loader.modificationCheckInterval">1</prop>
                <prop key="velocimacro.library.autoreload">true</prop>
                <prop key="velocity.engine.resource.manager.cache.enabled">false</prop>
                <prop key="springMacro.resource.loader.cache">false</prop>
                <prop key="eventhandler.referenceinsertion.class">
                    org.apache.velocity.app.event.implement.EscapeXmlReference
                </prop>
            </props>
        </property>
    </bean>

#### 参考 {#ref}

[1]<http://stackoverflow.com/questions/7414794/difference-between-contextannotation-config-vs-contextcomponent-scan>

[2]<http://stackoverflow.com/questions/7621920/scopeprototype-bean-scope-not-creating-new-bean>
