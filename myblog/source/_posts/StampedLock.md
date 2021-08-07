---
title: StampedLock
categories:
  - JUC
  - StampedLock
tags: StampedLock
abbrlink: ab957bc5
date: 2021-03-25 22:52:21
---
##### 由来

> ReentrantReadWriteLock 和 StampedLock完成的功能相似，但是StampedLock相比ReentrantReadWriteLock 提升了程序的性能和吞吐量。改进的点在于
>
> * ReentrantReadWriteLock 在沒有任何读写锁时，才可以取得写入锁，这可用于实现了悲观读取(Pessimistic Reading)，即如果执行中进行读取时，经常可能有另一执行要写入的需求，为了保持同步，ReentrantReadWriteLock 的读取锁定就可派上用场。
>
>   然而，如果读取执行情况很多，写入很少的情况下，使用 ReentrantReadWriteLock 可能会使写入线程遭遇饥饿(Starvation)问题，也就是写入线程迟迟无法竞争到锁定而一直处于等待状态。
>
>   而 StampedLock的乐观读锁则解决了这一问题，**乐观读锁认为读的时候是不会有写锁产生的**

<!--more-->

##### 官网定义

>整体来说，StampedLock用三种模式来控制读写的访问(**写、读、乐观读**),锁的状态由版本和模式确定。锁获取方法会返回一个stamp,返回为0表示执行失败。
>锁的释放和转换会把stamp作为参数，如果它的值与状态不匹配则返回0表示失败！

##### Javadoc例

```java
class Point {
        private final StampedLock sl = new StampedLock();
        private double x, y;
		
    	//写锁
        void move(double deltaX, double deltaY) { // an exclusively locked method
            long stamp = sl.writeLock();
            try {
                x += deltaX;
                y += deltaY;
            } finally {
                sl.unlockWrite(stamp);
            }
        }

        //下面看看乐观读锁案例 乐观读锁认为读的时候是不会有写锁产生的
        double distanceFromOrigin() { // A read-only method
            long stamp = sl.tryOptimisticRead(); //获得一个乐观读锁
            double currentX = x, currentY = y; //将两个字段读入本地局部变量
            if (!sl.validate(stamp)) { //检查发出乐观读锁后同时是否有其他写锁发生?
                stamp = sl.readLock(); //如果没有，我们再次获得一个读悲观锁
                try {
                    currentX = x; // 将两个字段读入本地局部变量
                    currentY = y; // 将两个字段读入本地局部变量
                } finally {
                    sl.unlockRead(stamp);
                }
            }
            return Math.sqrt(currentX * currentX + currentY * currentY);
        }

        //下面是悲观读锁案例
        void moveIfAtOrigin(double newX, double newY) { // upgrade
            // Could instead start with optimistic, not read mode
            long stamp = sl.readLock();
            try {
                while (x == 0.0 && y == 0.0) { //循环，检查当前状态是否符合
                    long ws = sl.tryConvertToWriteLock(stamp); //将读锁转为写锁 这里锁升级了！
                    if (ws != 0L) { //这是确认转为写锁是否成功
                        stamp = ws; //如果成功 替换票据
                        x = newX; //进行状态改变
                        y = newY; //进行状态改变
                        break;
                    } else { //如果不能成功转换为写锁
                        sl.unlockRead(stamp); //我们显式释放读锁
                        stamp = sl.writeLock(); //显式直接进行写锁 然后再通过循环再试
                    }
                }
            } finally {
                sl.unlock(stamp); //释放读锁或写锁
            }
        }
    }
```

###### 写锁例子

`writeLock()`：获取写锁，如果加锁失败会一直获取直到能获取到为止，返回一个`stamp`，有可能会阻塞。使用`unlockWrite(long)`来释放锁。`tryWriteLock()`也是获取写锁，但获取失败会返回stamp=0.对应释放锁方法`tryUnlockWrite()`.此方法用于异常的恢复。

###### 乐观读例子

根据`tryOptimisticRead()`方法的注释

`Returns a stamp that can later be validated, or zero if exclusively locked.`

可以知道，

> 如果排他锁即写锁存在，返回的stamp为0  **乐观读锁认为读的时候是不会有写锁产生的**
>
> ```java
> @Test
>  public void testTryOptimisticRead() throws InterruptedException {
>      long stamp = sl.writeLock();
>      new Thread(()->{
>          System.out.println("写锁被占用后： "+sl.tryOptimisticRead());
>      }).start();
>      TimeUnit.SECONDS.sleep(2);//为了保证锁释放前被另一个线程调用
>      sl.unlockWrite(stamp);
> 
>      stamp = sl.readLock();
>      new Thread(()->{
>          System.out.println("读锁被占用后： "+sl.tryOptimisticRead());
>      }).start();
>      TimeUnit.SECONDS.sleep(2);
>      sl.unlockRead(stamp);
>  }
> ```
>
> 调用结果
>
> > 写锁被占用后： 0
> > 读锁被占用后： 512
>
> 获取乐观读锁的方法一般与`validate`方法一起使用，有以下三种情况
>
> * `Returns true if the lock has not been exclusively acquired since issuance of the given stamp.` 锁没被占用，即没有线程去加锁，则返回true
>
> * `Always returns false if the  stamp is zero.`
>
>   stamp为0（一般表示尝试加锁失败）返回false
>
> * `Always returns true if the stamp represents a currently held lock.`
>
>   当前线程持有锁，返回true
>
> * `Invoking this method with a value not obtained from {@link #tryOptimisticRead} or a locking method for this lock has no defined effect or result.`
>   
>   stamp 不是从`tryOptimisticRead()`或者加锁方法中得到的，则没有意义，返回false。

进入获取乐观读锁的方法实现

```java
public long tryOptimisticRead() {
        long s;
        return (((s = state) & WBIT) == 0L) ? (s & SBITS) : 0L;
    }
public boolean validate(long stamp) {
    U.loadFence();
    return (stamp & SBITS) == (state & SBITS);
}
```

这里使用了内存读屏障`U.loadFence()`，严格保证了该方法之前的所有load操作在内存屏障之前完成。

###### 读锁例子

`readLock()`：获取读锁，同样可能会造成阻塞，返回一个stamp.对应释放锁方法为`unlockRead(long)`。也有`tryReadLock`和`tryUnlockRead`方法，通常也是用于异常后的恢复。基本思想是，先加一个读锁，判断数据有没有被修改，没被修改则将读锁转为写锁，转为写锁成功则修改数据，否则释放读锁资源，再次获取写锁，直到获取成功为止。



##### 参考

1.  [stampedlock解析](https://www.pdai.tech/md/java/java8/java8-stampedlock.html )