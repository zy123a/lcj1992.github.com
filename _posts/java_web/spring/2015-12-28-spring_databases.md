---
layout: post
title: spring分库
categories: java_web
tags: spring 分库
---

#### db-context.xml

    <bean id="parentDatasource" class="xxxxx" destroy-method="close">
        ...
    </bean>
    
    <bean id="dataSource1" parent="parentDataSource">
        <property name="url" value="jdbc:mysql://localhost:3306/simple"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>
    
    <bean id="dataSource2" parent="parentPoolDataSource">
        <property name="url" value="jdbc:mysql://localhost:3306/simple2"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>
    
    <bean name="dynamicDataSource" class="com.xxx.web.util.DynamicDataSource">
        <property name="targetDataSources">
            <map key-type="java.lang.String">
                <entry value-ref="dataSource1" key="xxx1"/>
                <entry value-ref="dataSource2" key="xxx2"/>
            </map>
        </property>
    </bean>
        ...

#### DynamicDataSource.java

    public class DynamicDataSource extends AbstractRoutingDataSource {
        @Override
        protected Object determineCurrentLookupKey() {
            return AppContext.getContext();
        }
    }

#### AppContext.java

    public class AppContext(){
        private static InheritableThreadLocal<String> context = new InheritableThreadLocal<String>();
        
        public static void setContext(String context){        
            AppContext.context.set(context);    
        }
        
        public static String getContext(){        
            return AppContext.context.get();
        }
    }
