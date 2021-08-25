---
title: JVM
categories:
  - JVM
tags: JVM
abbrlink: 66c016fb
date: 2021-03-23 22:52:21
---

### JVM内存结构

<img src="http://www.caijy.top/image-20210318221702882.png" alt="image-20210318221702882" style="zoom: 80%;" />

#### 运行时数据区

# ![image-20210321125505092](http://www.caijy.top/image-20210321125505092.png)

##### 程序计数器

> 程序计数器，可以看作是当前线程所执行的字节码的行号指示器。
>
> 字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。
>
> 1. 为什么说程序计数器是线程私有的？
>
>    >为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储。
>    >
>    
>2. 如果线程正在执行的是一个**Java方法**，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是**本地（Native）方法**，这个计数器值则应为空（Undefined）。
> 

<!--more-->

##### 虚拟机栈

>* 虚拟机栈描述的是Java方法执行的线程内存模型: 每个方法被执行的时候，JAVA虚拟机都会创建一个**栈帧**用于存储**局部变量表、操作数栈、动态连接、方法出口**等信息。**方法从调用到执行结束，对应着栈帧在虚拟机栈中入栈到出栈的过程**。
>* 注意： 
>
>> 栈申请内存失败，会出现OutOfMemoryError异常
>
> HotSpot虚拟机的栈容量是不可以动态扩展的。只要线程申请栈空间成功了就不会有OOM，但是如果申请时就失败，仍然是会出现OOM异常的。
>
>> 线程请求的栈深度大于虚拟机允许的深度，会出现StackOverflowError异常

###### 局部变量表

> * 局部变量表存放的是**基本数据类型**、**对象引用**(它并不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或者其他与此对象相关的位置)和**returnAddress类型**（指向了一条字节码指令的地址）。
>
> * 局部变量表由**局部变量槽**组成，除了64位**long和double**类型的数据会占用**两个变量槽**外，其余的数据类型只占用一个。局部变量表所需的内存空间，在程序编译期间就已完成分配。

###### 操作数栈（Operand Stacks）

>每个栈帧都包含一个后进先出的操作数栈。当栈帧被创建时，一个空的操作数栈就被创建了。操作数栈会使用指令将局部变量表的常量或值加载进来。

###### 动态连接(Dynamic Linking)

> - 每一个栈帧内部都包含一个指向运行时常量池中该栈帧所属方法的引用。 包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接( Dynamic Linking）。比如: invokedynamic指令
> - 在Java源文件被编译到字节码文件中时，所有的变量和方法引用都作为符号引用（ symbolic Reference）保存在class文件的常量池里。 比如:描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么动态链接的作用就是为了将**这些符号引用转换为调用方法的直接引用**。

###### 方法出口

> * 当一个方法开始执行后，只有两种方式可以退出这个方法。方法退出的时候相当于当前栈帧出栈。
>   * 第一种是JVM碰到任意一个方法返回的字节码指令，被称为正常完成出口。
>   * 另一种是在执行方法中抛出异常并且未对异常进行处理，被称为异常完成出口。

##### 本地方法栈

> 本地方法栈为虚拟机使用到的本地(Native)方法服务。与虚拟机栈一样，本地方法栈也会在栈深度溢出或者栈扩展失败时分别抛出StackOverflowError和OutOfMemoryError异常。

##### 堆

> The heap is the runtime data area from which memory for all class instances and arrays is allocated。

《Java虚拟机规范》原文的意思是"所有的对象实例以及数组都应当在堆上分配。"但是随着现在编译技术的进步，HotSpot使用了**逃逸分析**技术，出现了**栈上分配、标量替换**等优化手段，这使得这句话不再这么绝对了。现在的垃圾回收器也不一定是分代垃圾回收器了，HotSpot也出现了不采用分代设计的新垃圾收集器。从分配内存的角度看，所有线程共享的Java堆中可以划分出多个线程私有的**分配缓冲区**（Thread Local Allocation Buffer，TLAB），以提升对象分配时的效率。

##### 方法区

> It stores per-class structures such as the run-time constant pool, field and method data, and the code for methods and constructors, including the special methods  used in class and instance initialization and interface initialization
>
> 《Java虚拟机规范》提到方法区存储的是运行时常量池，类的属性字段及方法，构造方法的缓存代码包括类和接口初始化的方法。
>
> 《深入理解JAVA虚拟机》提到"它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。"
>
> * java6之前，常量、静态变量、类元信息存储在永久代中，到了jdk6，HotSpot开发团队就有放弃永久代，逐步改为采用本地内存（Native Memory）来实现方法区的计划了。到了jdk7，原本放在永久代的字符串常量池和静态变量就被移出，到了jdk8才完全废弃了永久代的概念，永久代剩余的内容(类型信息)全部移到用本地内存实现的元空间中。
>
> * 总之，只要记住：方法区是由本地内存实现的，它存储的主要是运行时常量池、静态变量和类型信息。元空间是方法区的一部分，存储的主要是类型信息。
>
> * 那么疑问出现了，openjdk官网提到"The proposed implementation will allocate class meta-data in native memory and move interned Strings and class statics to the Java heap."意思是说，字符串常量和类静态数据都放到了java堆中。其实可以说，字符串常量和静态变量放在方法区。因为《Java虚拟机规范》把方法区描述为，逻辑上为堆的一部分。不过既然有了方法区这个概念把堆区分开来，那么说字符串常量和类静态数据都放到了方法区中，也是对的。
>   *  运行时常量池 
>
>     >对于运行时常量池，除了保存Class文件中描述的符号引用外，还会把由符号引用翻译出来的直接引用也存储在运行时常量池中。典型的例子就是 String类的intern()方法，下面是示例。
>     >
>     >1. 直接用双引号声明出来的 String对象 会直接分配在存储在字符串常量池里面。，`"" `就相当于` "".intern();`
>     >2. 如果不是用双引号声明的 String对象，可以调用String对象的intern方法。intern方法会检查字符串常量池里面是否存在当前字符串，如果不存在就会把该字符串存储在常量池里，存在则直接返回该字符串的引用。
>     >
>     >  ```java
>     >   /*
>     >    * 常量池中的串池StringTable[],是一个哈希表
>     >    * String intern的插入逻辑：先判断是否这个值 没有的话就插入
>     >    */
>     >   public class Test {
>     >       public static void main(String[] args) {
>     >           String a = "ab";//StringTable  ["ab"]
>     >           String ab = "a"+"b";//本质是使用StringBuilder拼接，此时串池中已有ab 直接引用
>     >           System.out.println(a == ab);// true
>     > 
>     >           String s = new String("1");//执行后 s会被放入堆中
>     >           String intern = s.intern();//串池中为 ["ab"],没有"1"所以需要加入串池。执行完 ["ab","1"]
>     >           String s2 = "1";
>     >           System.out.println(s == s2);// false s在堆中 s2引用的是串池中的值 不相等
>     >           System.out.println(intern == s2);// true
>     > 
>     >           String s3 = new String("X") + new String("Y");//执行完 s3会放入堆中
>     >           s3.intern();//StringTable中没有"XY"，插入后为  ["ab","1","XY"]
>     >           String s4 = "XY";
>     >           System.out.println(s3 == s4);// true
>     >       }
>     >   }
>     >  ```



### 类加载子系统

###### 