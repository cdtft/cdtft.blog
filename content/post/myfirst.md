---
title: "单例延迟初始化"
date: 2020-03-23T14:40:37+08:00
tags: ["java"]
---
## 类会何时被初始化
有一下几种情况类会初始化
1. 虚拟机启动的时候，初始化用户指定的主类
2. 遇到new
3. 调用静态方法时，初始化该静态方法所在的类
4. 访问静态字段，初始化该静态字段所在的类
5. 子类的初始化触发父类的初始化
6. `定义了default接口，直接实现或间接实现该接口的类初始化会触发该接口的初始化`
7. 使用反射API对某个类反射调用时
8. 初次调用Methodhandle实例时，初始化Methodhandle指向的方法所在的类

## 使用第4条规范实现单例模式
```java
public class Singleton {
    
    private Singleton() {
    }
    
    private static class LazyHolder {
        private final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
       return LazyHolder.INSTANCE; 
    }
    
}
```
该单例是线程安全的
