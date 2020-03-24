---
title: "Guide to the Spring BeanFactory(翻译)"
date: 2020-03-24T14:35:22+08:00
tags: ["Spring"]
---
# 1.Introduction
这篇文章将关注`Spring BeanFactory API`
`BeanFactory`接口提供了简单也可以说是复杂的配置机制通过Spring Ioc容器来管理对象。

# 2.Basic - Beans and Container
简单的说，beans是构成Spring application骨架的java对象，它们被Spring Ioc容器管理。除了用容器管理以外，bean没有什么其他特别的地方。

Spring容器的职责是实例化、配置、组装beans。Spring容器通过阅读配置元信息来实例化、配置、管理我们在程序中定义的对象。

# 3.Maven Dependencies
让我们添加需要的依赖在pom.xml文件中。我们将用Spring Beans依赖去设置BeanFactory:
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>5.1.4.RELEASE</version>
</dependency>
```

# 4.The BeanFactory Interface
一开始查看org.springframework.beans.factory包中的接口的定义和讨论重要的APIs是很有趣的事情。

## 4.1The getBean() APIs
各种版本的getBean()方法都返回了指定bean的实例

## 4.2The containsBean() APIs
这个方法通过提供的名称来确认bean工厂容器中是否存在该名称的bean。该方法确认getBean(java.lang.String)是否能获得给定名称的bean实例。

## 4.3The isPrototype() API
这个方法确认通过getBean(java.lang.String)方法获得实例是否被设置为prototype类型

需要注意的是如果该方法返回的的是false也不意味着是一个Singleton对象，也可能是是其他scopes non-independent实例。

# 5.BeanFactory API
BeanFactory持有bean的定义，并且当客户端程序请求时实例化 - 意思是：
* 关注bean实例化的生命周期，和在适当的时候调用销毁方法。
* 能够在实例化依赖对象之间建立关联
* 最重要的的是BeanFactory不支持基于注解配置的依赖注入，但是ApplicationContext则支持。

# 6.Defining the Bean
```java
public class Employee {
    private String name;
    private int age;
     
    // standard constructors, getters and setters
}
```
# 7.Configuring the BeanFactory with XML
我们可以使用XMl来配置BeanFactory,让我们创建beanfactory-exeample.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
 
  <bean id="employee" class="com.baeldung.beanfactory.Employee">
    <constructor-arg name="name" value="Hello! My name is Java"/>
    <constructor-arg name="age" value="18"/>
  </bean>    
  <alias name="employee" alias="empalias"/>
</beans>
```
# 8. BeanFactory with ClassPathResource
ClassPathResource 属于org.springframework.core.io包，让我们使用ClassPathResource快速测试和初始化XmlBeanFactory
