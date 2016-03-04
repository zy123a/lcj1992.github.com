---
layout: post
title: mybatis分表
categories: java_web
tags: mybatis 分表
---

#### Param注解

dao:将表名传入

    public int save(@Param("table") String table, @Param("name") String name, @Param("gender") Integer gender);

mapper.xml  
***"$"动态传入表名 "#"传入列值,传入map***

    <insert id="save" parameterType="Map">
        insert into ${table} (name,gender)values(#{name},#{gender})
    </insert>

#### Model

model:

    public class User {
        private Long id;
        private String table;
        private String name
        private Integer gender;
        ....getter setter
     }

dao:

    public int save(User user);

mapper.xml

    <insert id="save" parameterType="User">
        insert into ${table} (name,gender)values(#{name},#{gender})
    </insert>


Map