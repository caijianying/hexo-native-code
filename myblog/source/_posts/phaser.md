title: Phaser
categories:
  - JUC
  - Phaser
tags: JUC
date: 2021-03-26 22:52:21
---
##### 前言

> JDK5中引入了CyclicBarrier和CountDownLatch这两个并发控制类，而JDK7中引入的Phaser按照官方的说法是提供了一个功能类似但是更加灵活的实现。
>
> CyclicBarrier可以被看成一个发令枪，内部有个计数器，初始值为0。每个线程准备就绪后，计数器加一，直到等于设定的目标值。当线程全部就绪后，一起并发执行，类似于百米赛跑。
>
> CountDownLatch类似于下班后最后一个关门的人，等到最后一个人走了才把门关上。即线程全部执行完毕后在main线程中执行接下来的逻辑。
>
> CountDownLatch是一次性的，CyclicBarrier是可复用的。试想一下，如果**一个任务需要分多阶段进行，且每个阶段可以多个线程并发执行**，用CyclicBarrier和CountDownLatch实现就比较复杂了。为了解决这个问题，Phaser就登场了。
>
> Phaser 可以保证每个阶段执行的顺序，即阶段1必须执行完才会执行阶段2。

###### 例1

现在我们来讲一个寝室室友一起出去吃饭的例子。这个例子分为起床，穿衣洗漱，吃饭，散步，返回5个阶段。

<!--more-->

* 晒代码(假设有3个室友)

  ```java
  	public String[] name = new String[]{"pA","pB","pC"};
      public  Phaser phaser = new Phaser(name.length){
          @Override
          protected boolean onAdvance(int phase, int registeredParties) {
              System.out.println("*********阶段"+(phase+1)+"结束！********");
              return super.onAdvance(phase, registeredParties);
          }
      };
  
      class Person implements Runnable{
          private void getUp() {
              System.out.println(Thread.currentThread().getName()+"起床了！");
              phaser.arriveAndAwaitAdvance();
          }
          private void wash() {
              System.out.println(Thread.currentThread().getName()+"穿衣洗漱了！");
              phaser.arriveAndAwaitAdvance();
          }
          private void eat() {
              System.out.println(Thread.currentThread().getName()+"吃饭了！");
              phaser.arriveAndAwaitAdvance();
          }
  
          private void walk() {
              System.out.println(Thread.currentThread().getName()+"散步了！");
              phaser.arriveAndAwaitAdvance();
          }
          private void goHome() {
              System.out.println(Thread.currentThread().getName()+"回家了！");
              phaser.arriveAndAwaitAdvance();
          }
          @Override
          public void run() {
              getUp();
              wash();
              eat();
              walk();
              goHome();
          }
      }
  
      @org.junit.Test
      public void test01(){
          for (int i = 0; i < name.length; i++) {
              new Thread(new Person(),name[i]).start();
          }
      }
  ```

* 执行效果

  ###### ![image-20210331162417798](http://www.caijy.top//20210331162424.png)

* 类的说明

  ```java
   /**
   * @param parent   the parent phaser   父phaser
   * @param parties  the number of parties required to advance to the next phase  	       * 需要到达下一阶段的人数
   */
   public Phaser(Phaser parent, int parties) {
       //.....略
   }
  ```

  status是一个64位的long变量，设计十分巧妙，详见  [Phaser](http://www.voidcn.com/article/p-hvlsqvuf-bbe.html )。

  这里的源码就省略了，看了这段代码会发现，这个方法的实现用了大量的**位运算**去计算**status**的值，也使用了大量的**CAS**操作。考虑到并发条件下，status这个变量的资源会被大量竞争，CAS系统层面的操作很可能会被重复调用造成大量CPU开销。所以建议使用父Phaser，把对status一部分的计算分担到子Phaser中，父Phaser 只需负责推动每个阶段的进行，提高Phaser的吞吐量。

  ###### 常用的方法

  > `bulkRegister(int parties)`    批量注册需要协作的线程。
  >
  > `arriveAndAwaitAdvance()`         线程执行结束。必须等待其他线程
  >
  > `onAdvance(int phase, int registeredParties)`     每个阶段线程都执行完毕会调用此方法

  

###### 例2

```java
	public String[] name = new String[]{"pA", "pB", "pC", "pD", "pE"};
	public String[] nameX = new String[]{"XA", "XB", "XC", "XD", "XE"};
	@Test
    public void test02() {
        final Phaser parent = new Phaser() {
            @Override
            protected boolean onAdvance(int phase, int registeredParties) {
                System.out.println("*******parent***" + phase + "********");
                return super.onAdvance(phase, registeredParties);
            }
        };
        final Phaser phaser1 = new Phaser(parent) {
            @Override
            protected boolean onAdvance(int phase, int registeredParties) {
                System.out.println("*******phaser1***" + phase + "********");
                return super.onAdvance(phase, registeredParties);
            }
        };
        final Phaser phaser2 = new Phaser(parent) {
            @Override
            protected boolean onAdvance(int phase, int registeredParties) {
                System.out.println("*******phaser2***" + phase + "********");
                return super.onAdvance(phase, registeredParties);
            }
        };
        phaser1.bulkRegister(5);
        phaser2.bulkRegister(4);
        for (int i = 0; i < name.length; i++) {
            int finalI = i;
            Thread t = new Thread(() -> {
                System.out.println(name[finalI] + ":" + finalI);
                phaser1.arriveAndAwaitAdvance();
            });
            t.start();
        }
        for (int i = 0; i < 4; i++) {
            int finalI = i;
            Thread t = new Thread(() -> {
                System.out.println(nameX[finalI] + ":" + finalI);
                phaser2.arriveAndAwaitAdvance();
            });
            t.start();
        }
        parent.awaitAdvance(parent.getPhase());
    }
```

##### ![image-20210331165607204](http://www.caijy.top//20210331165607.png)

参考

1.  [Phaser使用和解析](http://www.voidcn.com/article/p-hvlsqvuf-bbe.html)