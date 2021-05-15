---
title: Jclasslib分析字节码
date: 2021-03-24 22:52:21
categories: 
- JVM
- 分析字节码
tags: JVM字节码
---

#### 下载及使用

    1. 搜索  "jclasslib Bytecode viewer "下载插件，并重启IDEA
    2. IDEA   view -> show Bytecode With Jclasslib

#### 上代码

```java
  @org.junit.Test
    public void test02() {
        int a = 10;
        int b = 20;
        int c = sum(a,b);
        System.out.println(c);
    }

    private int sum(int args1, int args2) {
        Lists.newArrayList().forEach((t)-> System.out.println(t));//为了能看到动态连接的指令
        return args1 + args2;
    }
```

##### 首先会将当前这个实例放入局部变量表

##### <img src="http://qqcxdmsd8.hn-bkt.clouddn.com/image-20210323224206973.png" style="zoom: 50%;" />

##### 解析字节码

注意：这里说的栈是指**操作数栈**。

###### `test02`的字节码

* > 0 bipush 10    //将常量10推送至栈顶
  > 2 istore_1       // 将int类型数值(10)存入局部变量表
  > 3 bipush 20   //将常量20推送至栈顶
  > 5 istore_2      // 将int类型数值(20)存入局部变量表
  > 6 aload_0      // 将第一个引用类型变量（this）推送至栈顶
  > 7 iload_1      // 将第二个int 型本地变量(a)推送至栈顶
  > 8 iload_2      // 将第二个int 型本地变量(b)推送至栈顶
  > 9 invokespecial #2 <Test2.sum>  //调用超类构造方法，实例初始化方法，私有方法
  >12 istore_3   // 将int类型数值(30)存入局部变量表
  >13 getstatic #3 <java/lang/System.out>
  >16 iload_3
  >17 invokevirtual #4 <java/io/PrintStream.println>
  >20 return  //当前方法返回void

###### `sum`的字节码 `

* > 0 invokestatic #5 <org/assertj/core/util/Lists.newArrayList>  
  > 3 invokedynamic #6 <accept, BootstrapMethods #0>   //**java8的lambda表达式 成为了 动态连接指令invokedynamic 的生成方式**
  > 8 invokevirtual #7 <java/util/ArrayList.forEach>
  >11 iload_1
  >12 iload_2
  >13 iadd     //将栈顶2个int类型的值相加并将结果压入栈顶
  >14 ireturn    //当前方法返回int

![image-20210405214141423](https://qqcxdmsd8.hn-bkt.clouddn.com/image-20210405214141423.png)

