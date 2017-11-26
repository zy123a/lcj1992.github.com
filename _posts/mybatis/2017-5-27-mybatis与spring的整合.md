---
layout: post
title: mybatis与spring的整合
tags: mybatis spring
categories: mybatis
---    

* TOC
{:toc}   

### 1、相关jar包引入    
```xml
 <!--spring mvc 集成mybatis-->
    <!--mybatis 核心jar-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.4</version>
    </dependency>
    <!--spring -mybatis 插件-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.1</version>
    </dependency>
    <!--mysql 连接-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>6.0.6</version>
    </dependency>
    <!--数据源-->
    <dependency>
      <groupId>c3p0</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.1.2</version>
    </dependency>
```    

### 2、spring的DataSource相关配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns="http://www.springframework.org/schema/beans" xmlns:tx="http://www.springframework.org/schema/tool"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	   http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tool http://www.springframework.org/schema/tool/spring-tool.xsd">
       // 数据源配置
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test"/>
        <property name="user" value="root"/>
        <property name="password" value="134138"/>
    </bean>   
    
    // 配置SqlSessionFactory用来产生SqlSession
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        // mybatis相关配置
        <property name="configLocation" value="classpath:maybatis-map-config.xml"/>
        // 相关mapper.xml
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
    </bean>
    
    //  封装了SqlSession的相关操作
    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
    
    // 用于扫描相关的mapperInterface.java的接口
    <bean id="scanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.zy.inferance"/>
    </bean>

    // 为数据源配置事务
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

     // 配置声明式事务
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
</beans>
```     

### 3、mybatis相关配置  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="cacheEnabled" value="false"/>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="proxyFactory" value="CGLIB"></setting>
    </settings>
    <typeAliases>
        <typeAlias type="com.zy.beans.Student" alias="student"/>
        <typeAlias type="com.zy.beans.Course" alias="course"/>
    </typeAliases>

    <typeHandlers>
        <typeHandler handler="com.zy.typehandle.MyStringTypeHandle" javaType="string" jdbcType="VARCHAR"/>
    </typeHandlers>

</configuration>
```    

### 4、接口文件   
文件位置：com.zy.interface    
```java
public interface CourseMapper {
    public List<Course> getCourseByStudentId(@Param("studentId") String studentId);
}

public interface StudentInferace {
    public Student selectById(int id);

    public List<Student> getStudentByName(@Param("name") String name);

    public List<Student> getStudentByMap(Map map);

    public List<Student> getStudentByPhone(@Param("phone") String phone);

    public int insertStudent(Student student);

}
```   

### 5、mapper.xml文件  
 **courseMapper.xml**    
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zy.inferance.CourseMapper">

    <sql id="all_clonum">
        id,studentId,course,core
    </sql>

    <select id="getCourseByStudentId" resultType="course">
        select <include refid="all_clonum"/> from course where studentId=#{studentId}
    </select>
</mapper>
```    

**student.xml**    
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zy.inferance.StudentInferace">

    <sql id="all_clonum">
        id,name,phone
    </sql>

    <select id="selectById" parameterType="int" resultMap="studentMap2">
        select <include refid="all_clonum"/> from student where id=#{id}
    </select>

    <select id="getStudentByName" resultType="student">
        SELECT id,name,phone FROM student where name like concat(#{name, javaType=string, jdbcType=VARCHAR, typeHandler=com.zy.typehandle.MyStringTypeHandle},'%');
    </select>

    <select id="getStudentByMap" parameterType="map" resultType="student">
        SELECT id,name,phone from student where name like concat(#{name},'%') and
        phone like concat(#{phone},'%')
    </select>

    <resultMap id="studentMap" type="student">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="phone" column="phone"/>
    </resultMap>

    <resultMap id="studentMap2" type="student">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="phone" column="phone"/>
        <collection property="courses" column="id" select="com.zy.inferance.CourseMapper.getCourseByStudentId"/>
    </resultMap>

    <select id="getStudentByPhone"  resultMap="studentMap">
        SELECT id,name,phone from student where phone like concat(#{phone},'%')
    </select>

    <insert id="insertStudent" parameterType="student" keyProperty="id" useGeneratedKeys="true">
        insert into student(name,phone) VALUES (#{name},#{phone})
    </insert>
</mapper>
```