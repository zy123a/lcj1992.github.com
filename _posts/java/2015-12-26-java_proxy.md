---
layout: post
title: 动态代理和cglib代理
categories: java
tags: proxy cglib
---

*   [动态代理](#dynamic_proxy)
*   [cglib代理](#cglib_proxy)

<h3 id="dynamic_proxy">动态代理</h3>

    只对实现接口的类有效 //todok

    BookFacade.java

    public interface BookFacade {
        public void addBook();
    }
    BookFacadeImpl.java

    public class BookFacadeImpl implements BookFacade {
        @Override
        public void addBook() {
            System.out.println("增加图书方法.....");
        }
    }
    BookFacadeProxy.java

    //实现java的动态代理只需要实现InvocationHandler接口就行
    public class BookFacadeProxy implements InvocationHandler {

        private Object target;

        public Object bind(Object target){
            this.target = target;
        //运行时创建一个实现了BookFacade接口的类,只对实现接口的类有效，这也是java动态代理的其缺点。
            return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                    target.getClass().getInterfaces(), this);
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            Object  result = null;
            System.out.println("事务开始");
            result = method.invoke(target,args);
            System.out.println("事务结束");
            return result;
        }
    }

<h3 id="cglib_proxy">cglib代理</h3>

    BookFacadeImpl1.java

    public class BookFacadeImpl1 {
        public void addBook(){
            System.out.println("增加图书的普通方法...");
        }
    }
    BookFacadeCglib.java

    public class BookFacadeCglib implements MethodInterceptor {
        private Object target;

        public Object getInstance(Object target){
            this.target = target;
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(this.target.getClass());
            enhancer.setCallback(this);
            return enhancer.create();
        }

        @Override
        public Object intercept(Object obj, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
            System.out.println("事务开始");
            methodProxy.invokeSuper(obj, objects);
            System.out.println("事务结束");
            return null;
        }
    }
