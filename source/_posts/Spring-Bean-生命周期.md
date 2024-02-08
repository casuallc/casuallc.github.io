---
title: Spring Bean 生命周期
date: 2024-02-07 09:33:00
author: changqing
tags: 
- 教程
- 学习
- Spring Boot
categories:
- 2024
---

# Spring Bean 生命周期
spring 在初始化时会通过注解或者配置文件初始化所有的 bean，在初始化过程中支持执行自定义的逻辑，比如加载配置等。
spring bean 的生命周期如下图所示：
![Spring Bean 生命周期](/imgs/2024/0207_spring-bean-life.png)

<!-- more -->

其中 BeanPostProcessor 是对所有 Bean 进行拦截， InitializingBean 是对当前 Bean 进行拦截。所以如果需要在一个 Bean 初始化时做一些初始化的工作，则要实现 InitializingBean 接口。
示例如下
``` java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;

@Component
public class TestBeanInit implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }
}
```
实现该接口后会在每个 Bean 初始化时都进行拦截，因此会传递当前初始化的 Bean。

```java
import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Component;

@Component
public class TestBeanInit implements InitializingBean {

    private String name;

    @Override
    public void afterPropertiesSet() throws Exception {
        name = "123";
    }
}
```
该接口只会在当前 Bean 初始化时执行。