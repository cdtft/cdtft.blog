---
title: "How to use the Spring FactoryBean?(翻译)"
date: 2020-03-24T13:50:44+08:00
tags: ["Spring"]
---
# 1. Overview
有两种类型的bean在Spring bean容器中：普通的bean(ordinary bean)和工厂bean(factory bean)。Spring直接使用前者，后者能够产生又Spring管理的对象。简单的说，我们可以通过实现org.springframework.beans.factory.FactoryBean接口来构建一个工程bean。
# 2. Factory Beans基础
## 2.1 实现一个FactoryBean
首先来让我们看看FactoryBean接口
```java
public interface FactoryBean {
    T getObject() throws Exception;
    Class<?> getObjectType();
    boolean isSingleton();
}
```
让我们详细的看看这个三个方法的功能
* getObject()-返回一个由该工厂产生对象，这个对象将被Spring容器使用
* getObjectType()-返回该工厂生产bena对象的类类型
* isSingleton()-判断由该工厂产生的bean是否是单例的
接下来我们实现一个FactoryBean实例。实现一个产生Tool类型的ToolFactory。
```java
public class Tool {
 
    private int id;
 
    // standard constructors, getters and setters
}
```
ToolFactory.java:
```java
public class ToolFactory implements FactoryBean<Tool> {
 
    private int factoryId;
    private int toolId;
 
    @Override
    public Tool getObject() throws Exception {
        return new Tool(toolId);
    }
 
    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }
 
    @Override
    public boolean isSingleton() {
        return false;
    }
 
    // standard setters and getters
}
```
我们可以看出来ToolFactory是一个产生Tool对象的FactoryBean。
## 2.2 基于XML配置使用FactoryBean
让我们来看看如何使用ToolFactory
我们先构造一个基于XML配置的tool - factorybean-spring-ctx.xml:
```xml
<beans ...>
 
    <bean id="tool" class="com.baeldung.factorybean.ToolFactory">
        <property name="factoryId" value="9090"/>
        <property name="toolId" value="1"/>
    </bean>
</beans>
```
接下来测试Tool对象是否能注入：
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:factorybean-spring-ctx.xml" })
public class FactoryBeanXmlConfigTest {
    @Autowired
    private Tool tool;
 
    @Test
    public void testConstructWorkerByXml() {
        assertThat(tool.getId(), equalTo(1));
    }
}
```
测试结果表明，我们使用在factorybean-spring-ctx.xml中配置的属性注入由ToolFactory生成的tool对象。
测试结果也显示，Spring容器FactoryBean产生的对象进行依赖注入而不是对象自生。
Spring容器使用FactoryBean的getObject()方法返回该工厂产生的bean，也可以返回工厂本身。
`想要获取FactoryBean，仅仅需要在bena name前加上'&'符号`

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:factorybean-spring-ctx.xml" })
public class FactoryBeanXmlConfigTest {
 
    @Resource(name = "&tool")
    private ToolFactory toolFactory;
 
    @Test
    public void testConstructWorkerByXml() {
        assertThat(toolFactory.getFactoryId(), equalTo(9090));
    }
}
```
## 2.3 使用基于java配置的FactoryBean
使用FacotyBean通过java配置和基于XML配置有一点小的差别，java配置需要明确的调用FactotyBean的getObject()方法。
```java
@Configuration
public class FactoryBeanAppConfig {
  
    @Bean(name = "tool")
    public ToolFactory toolFactory() {
        ToolFactory factory = new ToolFactory();
        factory.setFactoryId(7070);
        factory.setToolId(2);
        return factory;
    }
 
    @Bean
    public Tool tool() throws Exception {
        return toolFactory().getObject();
    }
}

```
接下来测试Tool是否能被正确注入：
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = FactoryBeanAppConfig.class)
public class FactoryBeanJavaConfigTest {
 
    @Autowired
    private Tool tool;
  
    @Resource(name = "&tool")
    private ToolFactory toolFactory;
 
    @Test
    public void testConstructWorkerByJava() {
        assertThat(tool.getId(), equalTo(2));
        assertThat(toolFactory.getFactoryId(), equalTo(7070));
    }
}
```
这和之前XML配置测试的效果一样。
# 3.初始化方式
在有些时候我们需要在FactoryBean设置之后，getObject()方法调用之前做一些操作例如属性检查。
你可以实现InitializingBean接口或者使用@PostConstruct注解。

# 4.AbstractFactoryBean
Spring提供了一个实现FactoryBean接口的模版超类AbstractFacortyBean。通过这个基础类我们可以很方便的实现产生singleton和prototype类型bean的工厂bean。
让我们实现SingleToolFactory和NonSingleToolFactory来展示如何使用AbstractFactoryBean来产生singleton和prototype类型的bean。
```
public class SingleToolFactory extends AbstractFactoryBean<Tool> {
 
    private int factoryId;
    private int toolId;
 
    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }
 
    @Override
    protected Tool createInstance() throws Exception {
        return new Tool(toolId);
    }
 
    // standard setters and getters
}
```
nonsingleton实现：
```
public class NonSingleToolFactory extends AbstractFactoryBean<Tool> {
 
    private int factoryId;
    private int toolId;
 
    public NonSingleToolFactory() {
        setSingleton(false);
    }
 
    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }
 
    @Override
    protected Tool createInstance() throws Exception {
        return new Tool(toolId);
    }
 
    // standard setters and getters
}
```
factory bean的XML配置：
```xml
<beans ...>
 
    <bean id="singleTool" class="com.baeldung.factorybean.SingleToolFactory">
        <property name="factoryId" value="3001"/>
        <property name="toolId" value="1"/>
    </bean>
 
    <bean id="nonSingleTool" class="com.baeldung.factorybean.NonSingleToolFactory">
        <property name="factoryId" value="3002"/>
        <property name="toolId" value="2"/>
    </bean>
</beans>
```
我们可以测试工作的对象属性注入是否和我们期望的一样。
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:factorybean-abstract-spring-ctx.xml" })
public class AbstractFactoryBeanTest {
 
    @Resource(name = "singleTool")
    private Tool tool1;
  
    @Resource(name = "singleTool")
    private Tool tool2;
  
    @Resource(name = "nonSingleTool")
    private Tool tool3;
  
    @Resource(name = "nonSingleTool")
    private Tool tool4;
 
    @Test
    public void testSingleToolFactory() {
        assertThat(tool1.getId(), equalTo(1));
        assertTrue(tool1 == tool2);
    }
 
    @Test
    public void testNonSingleToolFactory() {
        assertThat(tool3.getId(), equalTo(2));
        assertThat(tool4.getId(), equalTo(2));
        assertTrue(tool3 != tool4);
    }
}
```
通过测试我们可以看到SingleToolFactory产生singleton对象，NonSingleToolFactory产生prototype对象。
注意无需设置singleton属性在SingleToolFactory中因为在AbstractFactory中singleton熟悉的默认值就是true


