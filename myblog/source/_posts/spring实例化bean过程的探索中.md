---
title: spring实例化bean的过程探索(下)
categories:
  - spring 5.1.X源码
tags: spring源码
abbrlink: '7791370'
date: 2021-07-28 22:52:21
---

# 前言

* 在上篇已经知道Spring的bean是从单例池中取的，但是有两个问题依旧是个迷！
  * 1. Spring的bean是如何被创建的？
    2. 创建好的bean是怎么被放进单例池的？



# 探索

* 由上篇可以推断出Spring容器在初始化的时候就已经创建好了bean，所以我们从第一次取出`testBean`为空的地方开始打断点，看看它接下来做了什么。下面开始探索。

![spring创建bean源码调试1](https://www.caijy.top//spring%E5%88%9B%E5%BB%BAbean%E6%BA%90%E7%A0%81%E8%B0%83%E8%AF%952-1.gif)

* 录屏比较长，跟着调试会知道：Spring会从`beanDefinition`中推断出构造方法，通过构造方法new一个实例出来，核心代码段是`ctor.newInstance(args)`。接着会填充属性，执行了`addSingleton(beanName, singletonObject)`方法，将完整的bean保存在单例池中。