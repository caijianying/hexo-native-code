---
title: spring实例化bean的过程探索(上)
categories:
  - spring 5.1.X源码
tags: spring源码
abbrlink: '1e622231'
date: 2021-07-26 22:52:21
---

# 准备工作

* 采用注解方式或者XML方式，这里使用的是注解方式

* 创建一个简单的全局配置类，指定包扫描路径即可

  ```java
  package com.debug.config;
  
  import org.springframework.context.annotation.ComponentScan;
  import org.springframework.context.annotation.Configuration;
  
  @Configuration
  @ComponentScan("com.debug")
  public class AppConfig {
    
  }
  ```

* 在扫描路径下创建一个class 

  ```java
  package com.debug.bean;
  
  import lombok.extern.slf4j.Slf4j;
  import org.springframework.stereotype.Component;
  
  @Slf4j
  @Component
  public class TestBean {
  	public TestBean() {
  		log.info("TestBean created ...");
  	}
  }
  ```

* 定义一个测试类Test

  ```java
  import com.debug.bean.TestBean;
  import com.debug.config.AppConfig;
  import lombok.extern.slf4j.Slf4j;
  import org.junit.Test;
  import org.springframework.context.annotation.AnnotationConfigApplicationContext;
  
  @Slf4j
  public class DebugTest {
  
  	@Test
  	public void testCrateBean(){
  		AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
  		TestBean bean = applicationContext.getBean(TestBean.class);//比较常用的getBean方法
  		log.info(bean.toString());
  	}
  }
  ```

* 运行结果

  <img src="https://www.caijy.top//%E6%88%AA%E5%B1%8F2021-07-26%20%E4%B8%8B%E5%8D%881.02.38.png" alt="截屏2021-07-26 下午1.02.38" style="zoom:50%;" />

<!--more-->

# 探索开始

## 第一次调试

![spring创建bean源码调试1](https://www.caijy.top//spring%E5%88%9B%E5%BB%BAbean%E6%BA%90%E7%A0%81%E8%B0%83%E8%AF%951.gif)

1. 经过第一波调试后发现重点在`DefaultListableBeanFactory`提供的`resolveNamedBean`,这里面包含了获取bean的步骤

## 第二次调试

![spring创建bean源码调试2](https://www.caijy.top//spring%E5%88%9B%E5%BB%BAbean%E6%BA%90%E7%A0%81%E8%B0%83%E8%AF%952.gif)

2. 第二次调试也有了进展，最终定位在了`AbstractBeanFactory`的`getBean`方法

## 第三次调试

![spring创建bean源码调试3](https://www.caijy.top//spring%E5%88%9B%E5%BB%BAbean%E6%BA%90%E7%A0%81%E8%B0%83%E8%AF%953.gif)

3. 第三次调试发现bean是从单例池中取的
   * 这说明容器对象`AnnotationConfigApplicationContext`初始化的时候，就已经将`testBean`实例化放在了`singletonObjects`中。
   * 那么`testBean`是怎么实例化的呢？这需要继续调试`AnnotationConfigApplicationContext`初始化的代码了。