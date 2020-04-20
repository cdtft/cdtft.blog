---
title: "BeanFactory的使用"
date: 2020-04-20T14:04:17+08:00
tag: ["Spring", "BeanFactory"]
---
之前翻译过一篇BeanFactory和FactoryBean区别的文章，当一个Bean的初始化逻辑很
复杂的时候可以实现FactoryBean接口来定制Bean的初始化逻辑。
## 定义FactoryBean
bean的定义：
```java
public class Cat {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Cat.class.getSimpleName() + "[", "]")
                .add("name='" + name + "'")
                .toString();
    }
}
```
为Cat实现工厂bean:
```java
@Component
public class CatFactoryBean implements FactoryBean<Cat> {

    @Override
    public Cat getObject() throws Exception {
        Cat car = new Cat();
        car.setName("this is create by factory");
        return car;
    }

    @Override
    public Class<?> getObjectType() {
        return Cat.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```
注意`@Componment`注解，需要将CatFactoryBean交给Spring IOC容器进行管理。
## 问题
我在使用`@Autowired`注解注入Cat的使用没有问题可以得到单例的Cat实例。
```java
@RestController
@RequestMapping("/login")
public class LoginController extends AbstractBaseController {

    @Autowired
    private Cat car;

    @PostMapping
    public ResponseVO login(@RequestBody LoginVO loginVO) {
        if (!loginVO.getUsername().equals("admin") || !loginVO.getPassword().equals("admin")) {
            return returnBadResult("登陆失败");
        }
        loginVO.setToken("Ba " + loginVO.getUsername() + "-" + loginVO.getPassword());
        return returnOKResult(loginVO);
    }

    @GetMapping("/cat")
    public void InjectCar() {
        System.out.println(this.car);
    }
}
```
于是我换了一种方式获取Cat
```java
@SpringBootApplication
public class MimiApplication {

	public static void main(String[] args) {
		SpringApplication springApplication = new SpringApplication(MimiApplication.class);
		springApplication.addListeners(new MyEventPublishingRunListener());
		ConfigurableApplicationContext ctc = springApplication.run();
		Cat car = (Cat) ctc.getBean("car");
		System.out.println(car);
	}

}
```
程序启动的时候报如下错误：
```
Exception in thread "restartedMain" java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.boot.devtools.restart.RestartLauncher.run(RestartLauncher.java:49)
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No bean named 'cat' available
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBeanDefinition(DefaultListableBeanFactory.java:771)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getMergedLocalBeanDefinition(AbstractBeanFactory.java:1221)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:294)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199)
	at org.springframework.context.support.AbstractApplicationContext.getBean(AbstractApplicationContext.java:1105)
	at com.cdtft.mimi.MimiApplication.main(MimiApplication.java:17)
	... 5 more
```
没有找到Cat的BeanDefinition,这一开始让我感到了很不解。
## 思考
后面通过我的尝试和联想之的知识，在Spring IOC容器中管理了两种Bean一种是不同的bean另外一种是FactoryBean
当调用ApplicationContext#getBean方法的时候传入方法的名称，当名称以`&`符号开头直接获取对应的FactoryBean
我在上文中注册CatBeanFactory的时候@Conpoment注解中没有填写任何的值，Bean name会默认为catBeanFactory
所以在使用ApplicationContext#getBean方法获取Cat Bean实例时传入的Bean名称应该为catBeanFactory.

为什么@Autowired注解没有问题？是因为该注解默认的是按照类型来注入。
