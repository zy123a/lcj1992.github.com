---
layout: post
title: 注解
categories: java
tags: annotation java
---

#### java中内置的三种注解

`@Override`: 重写方法，或者实现接口
`@Deprecated`:不鼓励使用，很危险或者是有更好的替代。
`@SuppressWarnings`: 压制警告

#### 元注解
元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它annotation类型作说明。

1.@Target

表示该注解可以用于什么地方，可能的ElementType参数包括：
CONSTRUCTOR：构造器的声明
FIELD：域声明（包括enum实例）
LOCAL_VARIABLE:局部变量声明
METHOD：方法声明
PACKAGE：包声明
PARAMETER：参数声明
TYPE：类，接口（包括注解类型）或enum声明

2.@Retention

表示需要在什么级别保存该注解信息。可选的RetentionPolicy参数包括：
SOURCE：注解将被编译器丢弃
CLASS：注解在class文件中可用，但会被VM丢弃
RUNTIME：VM将在运行期也保留注解，因此可以通过反射机制读取注解的信息。

3.@Documented

将此注解包含在JavaDoc中

4.@Inherited

允许子类继承父类中的注解

Class，Method，Field等类都实现了AnnotatedElement接口

#### 注解元素可用的类型

八大基本类型
String
Class
enum
Annotaion
以上的数组
注解允许嵌套

#### 自定义注解

        //Age
        @Target({ ElementType.METHOD, ElementType.FIELD })
        @Retention(RetentionPolicy.RUNTIME)
        public @interface Age {
            public int value() default 0;
        }
        //Hometown
        @Target({ ElementType.METHOD, ElementType.FIELD })
        @Retention(RetentionPolicy.RUNTIME)
        public @interface Hometown {
            public String value() default "";
        }
        //Name
        @Target({ ElementType.METHOD, ElementType.FIELD })
        @Retention(RetentionPolicy.RUNTIME)
        public @interface Name {
            public String value() default "";
        }
        //User
        public class User {
            @Name(value = "lcj")
            private String name;

            @Hometown(value = "henan")
            private String hometown;

            @Age(value = 22)
            private Integer age;
            ...

            getter/setter
            @Override
            public String toString() {
                return MoreObjects.toStringHelper(User.class).add("name", this.getName()).add("hometown", this.getHometown())
                    .add("age", this.getAge()).toString();
            }
        }

        @Test
        public void getUserAnnotation() {
            User user = new User();
            Field[] fields = User.class.getDeclaredFields();
            for (Field field : fields) {
                field.setAccessible(true);
                if ("name".equals(field.getName())) {
                    Name name = field.getAnnotation(Name.class);
                    user.setName(name.value());
                    System.out.println(name.value());
                }
                if ("hometown".equals(field.getName())) {
                    Hometown hometown = field.getAnnotation(Hometown.class);
                    user.setHometown(hometown.value());
                    System.out.println(hometown.value());
                }
                if ("age".equals(field.getName())) {
                    Age age = field.getAnnotation(Age.class);
                    user.setAge(age.value());
                    System.out.println(age.value());
                } else {
                    continue;
                }
            }
            System.out.println(user);
        }
