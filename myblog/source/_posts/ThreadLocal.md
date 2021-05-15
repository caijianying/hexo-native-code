---
title: ThreadLocal不简单
date: 2021-03-24 22:52:21
categories: 
- ThreadLocal
tags: ThreadLocal与弱引用
---
##### 弱引用介绍

> 弱引用也是用来描述那些非必须对象，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生为止。当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。在JDK 1.2版之后提供了WeakReference类来实现弱引用。

周志明书中提到了弱引用的概念，其中最典型的是ThreadLocal的例子。下面我们来探究。

##### ThreadLocal

* 这是官方JDK对ThreadLocal类的解释。

>This class provides thread-local variables.  These variables differ from
>their normal counterparts in that each thread that accesses one (via its
> {@code get} or {@code set} method) has its own, independently initialized
> copy of the variable.  {@code ThreadLocal} instances are typically private
> static fields in classes that wish to associate state with a thread (e.g.,
>a user ID or Transaction ID).

​	大致意思就是说，这个类会为每个线程提供一个本地变量，通过set和get方法设置和获取。每个线程只能修改本线程的那个值。

<!--more-->

##### ThreadLocal的set

* 接下来我们来看set方法：

  ```java
  public void set(T value) {
          Thread t = Thread.currentThread();
          ThreadLocalMap map = getMap(t);
          if (map != null)
              map.set(this, value);
          else
              createMap(t, value);
      }
  ```

  ThreadLocal接收到value，转手就交给了**ThreadLocalMap**。ThreadLocal初始化的时候，会走`createMap`这个方法，初始化一个容量为16的Entry数组。

  ```java
  
          private static final int INITIAL_CAPACITY = 16;
  
          private Entry[] table;
  
          private int size = 0;
  
          private int threshold; // Default to 0
  
  void createMap(Thread t, T firstValue) {
          t.threadLocals = new ThreadLocalMap(this, firstValue);
      }
  //... ... 省略部分代码
  ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
              table = new Entry[INITIAL_CAPACITY];
              int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
              table[i] = new Entry(firstKey, firstValue);
              size = 1;
              setThreshold(INITIAL_CAPACITY);
          }
  ```

  这里ThreadLocal已经初始化完毕，看`map.set(this, value);`

  ```java
   private void set(ThreadLocal<?> key, Object value) {
  
              // We don't use a fast path as with get() because it is at
              // least as common to use set() to create new entries as
              // it is to replace existing ones, in which case, a fast
              // path would fail more often than not.
  
              Entry[] tab = table;
              int len = tab.length;
              int i = key.threadLocalHashCode & (len-1);
  
              for (Entry e = tab[i];
                   e != null;
                   e = tab[i = nextIndex(i, len)]) {
                  ThreadLocal<?> k = e.get();
  
                  if (k == key) {
                      e.value = value;
                      return;
                  }
  
                  if (k == null) {
                      replaceStaleEntry(key, value, i);
                      return;
                  }
              }
  
              tab[i] = new Entry(key, value);
              int sz = ++size;
              if (!cleanSomeSlots(i, sz) && sz >= threshold)
                  rehash();
          }
  ```

  最后核心代码是` tab[i] = new Entry(key, value);` 我们来看这个entry

  ```java
  static class Entry extends WeakReference<ThreadLocal<?>> {
              /** The value associated with this ThreadLocal. */
              Object value;
  
              Entry(ThreadLocal<?> k, Object v) {
                  super(k);
                  value = v;
              }
          }
  ```

  可以看出这个Entry继承了一个弱引用，我们传入的k(即`this`)被`super(k);`调用。我们跟进到`WeakReference`的**顶级父类**`Reference`，k成了弱引用。

  ```java
   Reference(T referent) {
          this(referent, null);
      }
  
      Reference(T referent, ReferenceQueue<? super T> queue) {
          this.referent = referent;
          this.queue = (queue == null) ? ReferenceQueue.NULL : queue;
      }
  ```

  这说明ThreadLocal类的引用链是：entry的key(弱引用)指向了强引用，那么垃圾回收的时候就不会回收这个entry中的key。执行结果也确实说明没有被回收。

  ```java
  ThreadLocal<Integer> local = new ThreadLocal<>();
  local.set(10);
  System.out.println(local.get());
  System.gc();
  System.out.println(local.get());
  
  //执行结果： 
  //10
  //10
  ```

  那么疑问就来了，ThreadLocal使用不当是会造成内存泄露的，在使用完后需要手动去关闭这个ThreadLocal的使用，正确姿势是`local.remove();`我们使用`local.remove();`代替`System.gc();`。执行结果说明，remove方法并不是只是单纯做了一个GC操作，那么remove是怎么实现垃圾回收的呢？

  ```java
  ThreadLocal<Integer> local = new ThreadLocal<>();
  local.set(10);
  System.out.println(local.get());
  local.remove();
  System.out.println(local.get());
  
  //执行结果
  //10
  //null
  ```

##### ThreadLocal的remove

  ```java
  public void remove() {
           ThreadLocalMap m = getMap(Thread.currentThread());
           if (m != null)
               m.remove(this);
       }
  ```

  主要把remove方法交给了ThreadLocalMap去做，继续跟进到ThreadLocalMap的remove。

  ```java
  private void remove(ThreadLocal<?> key) {
              Entry[] tab = table;
              int len = tab.length;
              int i = key.threadLocalHashCode & (len-1);
              for (Entry e = tab[i];
                   e != null;
                   e = tab[i = nextIndex(i, len)]) {
                  if (e.get() == key) {
                      e.clear();
                      expungeStaleEntry(i);
                      return;
                  }
              }
          }
  ```

  `e.clear();`是核心，调用了**顶级父类**`Reference`删除了Entry中的key。这段代码切断了这条引用链，删除了强引用的关联，导致虚引用此时无法跟强引用建立关联，这样的话垃圾回收器会直接回收。内部实现很简单。

  ```java
  public void clear() {
          this.referent = null;
      }
  ```

##### 验证

  接下来我们看这段代码就能看懂了。

  > 这里直接调用垃圾回收器回收是回收不掉的，因为弱引用与强引用**1**建立了关联。

  ```java
  WeakReference<Integer> reference = new WeakReference<Integer>(1);
  System.out.println(reference.get());
  System.gc();
  System.out.println(reference.get());
  
  ```

  调用结果

  ```java
  1
  1
  ```

  > 接下来我们使用`reference.clear();`把引用删除，弱引用立马就被回收了。

  ```java
  WeakReference<Integer> reference = new WeakReference<Integer>(1);
  System.out.println(reference.get());
  reference.clear();
  System.out.println(reference.get());
  ```

  调用结果

  ```java
  1
  null
  ```

  

  