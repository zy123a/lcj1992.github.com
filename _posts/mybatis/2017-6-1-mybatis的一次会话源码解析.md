---
layout: post
title: mybatis一次会话源码解析
tags: mybatis 源码
categories: mybatis
---    
[toc]  

##### 1.主要构件介绍  
   
* **SqlSessionFactoryBuilder：** 负责生产SqlSessionFactory的类；   
  
* **XMLConfigBuilder：**负责解析mybatis的配置文件xxx.xml生产configuration的；   

* **configuration：** 保存着mybatis的所有配置信息，如settings设置，typeAliases别名，environment环境，mapper映射器；    

* **SqlSessionFactory：** 负责生成SqlSession的工厂，其只是一个接口而不是实现类，为此mybatis为他提供了一个实现类DefaultSqlSessionFactory；  

* **SqlSession：** mybatis的顶层api，表示与数据库的会话，完成增删改查数据的功能，主要包含四大对象：   
    1、Executor：执行器，负责调度StatementHandler，ParameterHandler，resultHandler来完成一次sql操作；  
    2、StatementHandler：使用数据库的Statement（preparedStatement）来与数据库进行交互，其是四大对象的核心，起到承上启下的作用；   
    3、ParameterHandler：用于对sql的参数进行处理；   
    4、resultHandler：对数据库返回的数据进行封装集成的；    
       
##### 2.源码解析    

```java
package com.meituan.service.mobile.meilv.utils;

import com.meituan.service.mobile.meilv.dao.bean.EnvironmentDo;
import com.meituan.service.mobile.meilv.dao.bean.Type;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import java.io.InputStream;

/**
 * Desc:
 * ------------------------------------
 * Author:zhengyin@meituan.com
 * Date:2017/5/11
 * Time:11:24
 */
public class MySqlSessionFactoryUtils {
    public static void main(String[] args) {
        String resource = "mybatis-config.xml";
        InputStream inputStream = null;
        try {
            //            读取配置文件
            inputStream = Resources.getResourceAsStream(resource);
            //            创建SqlSessionFactory
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
            SqlSession sqlSession = sqlSessionFactory.openSession();
            //            执行查询语句
            EnvironmentDo environmentDo = sqlSession
                    .selectOne("com.meituan.service.mobile.meilv.dao.EnvironmentDao.updateEnv",
                            Type.Rhone_callback.getId());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```    
###### 2.1、创建SqlSessionFactory的过程    
   
获取配置文件流后通过SqlSessionFactoryBuilder.build(...)方法来创建SqlSessionFactory，下面我们来看下这个方法：   

```java
public class SqlSessionFactoryBuilder {
    
  public SqlSessionFactory build(InputStream inputStream) {
    return build(inputStream, null, null);
  }

 public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
       // 通过创建XMLConfigBuilder来解析配置文件，生成Configuration对象
      XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      // 通过XMLConfigBuilder.parse()方法解析配置文件信息来创建Configuration，然后根据Configuration对象
      // 创建SqlSessionFactory对象
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        inputStream.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }
  
 public SqlSessionFactory build(Configuration config) {
     // 通过configuration对象来创建DefaultSqlSessionFactory对象；
     return new DefaultSqlSessionFactory(config);
   }
}
```   

###### 2.2、**创建Configuration的过程**     
 
 mybatis通过配置文件流创建XMLConfigBuilder对象，然后解析配置文件生产Configuration对象，下面我们具体看下这个创建过程代码：   
     
 ```java
  public class XMLConfigBuilder extends BaseBuilder {
  
    private boolean parsed;
    private XPathParser parser;
    private String environment;
    private ReflectorFactory localReflectorFactory = new DefaultReflectorFactory();
  
    public XMLConfigBuilder(InputStream inputStream, String environment, Properties props) {
      this(new XPathParser(inputStream, true, props, new XMLMapperEntityResolver()), environment, props);
    }
  
    private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
      super(new Configuration());
      ErrorContext.instance().resource("SQL Mapper Configuration");
      this.configuration.setVariables(props);
      this.parsed = false;
      this.environment = environment;
      this.parser = parser;
    }
  
    public Configuration parse() {
      if (parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
      }
      parsed = true;
      parseConfiguration(parser.evalNode("/configuration"));
      return configuration;
    }
  
    private void parseConfiguration(XNode root) {
      try {
        //issue #117 read properties first
        propertiesElement(root.evalNode("properties"));
        Properties settings = settingsAsProperties(root.evalNode("settings"));
        loadCustomVfs(settings);
        typeAliasesElement(root.evalNode("typeAliases"));
        pluginElement(root.evalNode("plugins"));
        objectFactoryElement(root.evalNode("objectFactory"));
        objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
        reflectorFactoryElement(root.evalNode("reflectorFactory"));
        settingsElement(settings);
        // read it after objectFactory and objectWrapperFactory issue #631
        environmentsElement(root.evalNode("environments"));
        databaseIdProviderElement(root.evalNode("databaseIdProvider"));
        typeHandlerElement(root.evalNode("typeHandlers"));
        mapperElement(root.evalNode("mappers"));
      } catch (Exception e) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
      }
    }
  }
```


 