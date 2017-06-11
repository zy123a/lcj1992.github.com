---
layout: post
title: mybatis一次会话源码解析
tags: mybatis 源码
categories: mybatis
---    
[toc]  

#### 1.主要构件介绍  
   
* **SqlSessionFactoryBuilder：** 负责生产SqlSessionFactory的类；   
  
* **XMLConfigBuilder：**负责解析mybatis的配置文件xxx.xml生产configuration的；   

* **configuration：** 保存着mybatis的所有配置信息，如settings设置，typeAliases别名，environment环境，mapper映射器；    

* **SqlSessionFactory：** 负责生成SqlSession的工厂，其只是一个接口而不是实现类，为此mybatis为他提供了一个实现类DefaultSqlSessionFactory；  

* **SqlSession：** mybatis的顶层api，表示与数据库的会话，完成增删改查数据的功能，主要包含四大对象：   
    1、Executor：执行器，负责调度StatementHandler，ParameterHandler，resultHandler来完成一次sql操作；  
    2、StatementHandler：使用数据库的Statement（preparedStatement）来与数据库进行交互，其是四大对象的核心，起到承上启下的作用；   
    3、ParameterHandler：用于对sql的参数进行处理；   
    4、resultHandler：对数据库返回的数据进行封装集成的；    
       
#### 2.源码解析    

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
            // 读取配置文件
            inputStream = Resources.getResourceAsStream(resource);
            // 创建SqlSessionFactory
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
            SqlSession sqlSession = sqlSessionFactory.openSession();
            // 执行查询语句
            EnvironmentDo environmentDo = sqlSession
                    .selectOne("com.meituan.service.mobile.meilv.dao.EnvironmentDao.updateEnv",
                            Type.Rhone_callback.getId());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```    
##### 2.1、创建SqlSessionFactory的过程    
   
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

##### 2.1.1、**创建Configuration的过程**     
 
 mybatis通过配置文件流创建XMLConfigBuilder对象，XMLConfigBuilder会将配置文件信息转换成document对象，而XML的配置定义
 文件转换成XMLMapperEntityResolver对象，封装在XPathParser对象中。XPathParser对象提供了获取DOM节点信息的方法。   
 
 ```java
  public class XMLConfigBuilder extends BaseBuilder {
  
    private boolean parsed;
    private XPathParser parser;
    private String environment;
    private ReflectorFactory localReflectorFactory = new DefaultReflectorFactory();
  
    public XMLConfigBuilder(InputStream inputStream, String environment, Properties props) {
      // 封装配置文件信息和配置文件定义到XPathParser对象
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
  
    /**
    * 解析配置文件信息方法
    */
    public Configuration parse() {
      if (parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
      }
      parsed = true;
      // 具体提取配置文件的<configuration>节点信息，然后分别解析其子节点Node：Mapper，properties，plugins，setting等
      parseConfiguration(parser.evalNode("/configuration"));
      return configuration;
    }
  
    /**
    * 解析配置文件的各个子节点信息组装成configuration对象，下面将以解析插件配置信息为例
    */
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
    /**
    * 解析配置文件中的插件信息
    */
    private void pluginElement(XNode parent) throws Exception {
        if (parent != null) {
          for (XNode child : parent.getChildren()) {
            // 获取插件节点名称
            String interceptor = child.getStringAttribute("interceptor");
            // 获取节点属性
            Properties properties = child.getChildrenAsProperties();
            // 创建插件实例
            Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
            // 设置插件属性
            interceptorInstance.setProperties(properties);
            // 将该插件添加到插件责任链中
            configuration.addInterceptor(interceptorInstance);
          }
        }
      }
  }
```  

##### 2.2 打开SqlSession的过程   
```java
public class DefaultSqlSessionFactory implements SqlSessionFactory {
    private final Configuration configuration;

    public DefaultSqlSessionFactory(Configuration configuration) {
        this.configuration = configuration;
    }

    public SqlSession openSession() {
        return this.openSessionFromDataSource(this.configuration.getDefaultExecutorType(), (TransactionIsolationLevel)null, false);
    }
    
    /**
    * 打开SqlSession会话，
    * execType 执行器类别，类别分为三种：simple简单执行器，REUSE预处理语句执行器，BATCH批量执行器
    * level 事务隔离级别，隔离级别分为四种：read uncommitted，read committed，repeatable read，Serializable 
    * autoCommit 是否自动提交事务
    */
    private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
            Transaction tx = null;
    
            DefaultSqlSession var8;
            try {
                // 获取环境配置，数据源配置等
                Environment e = this.configuration.getEnvironment();
                
                // 创建事务工厂
                TransactionFactory transactionFactory = this.getTransactionFactoryFromEnvironment(e);
                
                // 对数据源创建事务
                tx = transactionFactory.newTransaction(e.getDataSource(), level, autoCommit);
                
                // 创建执行器
                Executor executor = this.configuration.newExecutor(tx, execType);
                
                // 创建默认SqlSession会话
                var8 = new DefaultSqlSession(this.configuration, executor, autoCommit);
            } catch (Exception var12) {
                this.closeTransaction(tx);
                throw ExceptionFactory.wrapException("Error opening session.  Cause: " + var12, var12);
            } finally {
                ErrorContext.instance().reset();
            }
    
            return var8;
        }
}

```
 ##### 2.3 SqlSession如何来执行一次查询    
 
 让我们分析下   
 ``
 EnvironmentDo environmentDo = sqlSession.selectOne("com.meituan.service.mobile.meilv.dao.EnvironmentDao.updateEnv",Type.Rhone_callback.getId());
 ``  
 先来看下内部方法的实现                                 
 ```java
public class DefaultSqlSession implements SqlSession {
    private Configuration configuration;
    private Executor executor;
    private boolean autoCommit;
    private boolean dirty;
    private List<Cursor<?>> cursorList;
    
    public <T> T selectOne(String statement, Object parameter) {
            List list = this.selectList(statement, parameter);
            if(list.size() == 1) {
                return list.get(0);
            } else if(list.size() > 1) {
                throw new TooManyResultsException("Expected one result (or null) to be returned by selectOne(), but found: " + list.size());
            } else {
                return null;
            }
    }   
    
    public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
            List var5;
            try {
                /**
                * 1.根据Statement Id，在mybatis 配置对象Configuration中查找和配置文件相对应的MappedStatement
                */
                MappedStatement e = this.configuration.getMappedStatement(statement);     
               
                // 调用执行器执行查询语句
                var5 = this.executor.query(e, this.wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
            } catch (Exception var9) {
                throw ExceptionFactory.wrapException("Error querying database.  Cause: " + var9, var9);
            } finally {
                ErrorContext.instance().reset();
            }
            return var5;
        }
    

}
```
MyBatis在初始化的时候，会将MyBatis的配置信息全部加载到内存中，使用org.apache.ibatis.session.Configuration实例来维护。使用者可以使用sqlSession.getConfiguration()方法来获取。MyBatis的配置文件中配置信息的组织格式和内存中对象的组织格式几乎完全对应的。上述例子中的
```xml
<select id="selectById" resultType="com.meituan.service.mobile.meilv.dao.bean.EnvironmentDo">
        SELECT id ,ip,name FROM test_env WHERE id=#{id}
</select>
```  
加载到内存中会生成一个对应的MappedStatement对象，然后会以key="com.meituan.service.mobile.meilv.dao.EnvironmentDao#selectById" ，value为MappedStatement对象的形式维护到Configuration的一个Map中。当以后需要使用的时候，只需要通过Id值来获取就可以了。

###### 2.3.1 Executor执行查询任务   
```java
public class SimpleExecutor extends BaseExecutor {
    
     public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
            Statement stmt = null;
    
            List var9;
            try {
                Configuration configuration = ms.getConfiguration();
                
                /** 
                * 1、根据既有的参数，创建StatementHandler对象来执行查询操作
                * StatementHandler分三种：SimpleStatementHandler，PreparedStatementHandler，CallableStatementHandler
                */
                StatementHandler handler = configuration.newStatementHandler(this.wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
                
                // 2、依据handler和statementLog创建java.sql.Statement
                stmt = this.prepareStatement(handler, ms.getStatementLog());
                
                // 3、调用StatementHandler.query()方法，返回List结果集
                var9 = handler.query(stmt, resultHandler);
            } finally {
                this.closeStatement(stmt);
            }
    
            return var9;
        }
        
        /**
        *  创建Statement对象方法
        */
        private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
             // 获取连接   
            Connection connection = this.getConnection(statementLog);
            
            // 创建预处理Statement
            Statement stmt = handler.prepare(connection, this.transaction.getTimeout());
            
            // 向Statement设置参数
            handler.parameterize(stmt);
            return stmt;
         }
}
```     

依据参数



 
 

 