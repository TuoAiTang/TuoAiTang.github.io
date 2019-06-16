---
layout:       post
title:        "从StampedLock来看乐观锁与悲观锁"
subtitle:     "从使用到实现"
date:         2019-5-20 23：00
catalog:      true
tags:
    - 并发编程
    - JDK源码   
---

# StampedLock

## 类结构

StampedLock没有直接使用aqs的同步器，而是自己维护了一个CLH队列。


![类结构](/img/aqs/stampedlock_struct.png)

其实stamped lock分为三种模式。

- Read
- Write
- OptimisticRead

并且锁可以在持有期间升级。

- tryConvertToReadLock()
- tryConvertToOptimisticRead()
- tryConvertToWriteLock()

## 使用场景

```java
class Point {
    private double x, y;
    private final StampedLock sl = new StampedLock();

    void move(double deltaX, double deltaY) { // an exclusively locked method
        long stamp = sl.writeLock();
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            sl.unlockWrite(stamp);
        }
    }

    //乐观读
    double distanceFromOrigin() {
        //获得一个乐观读锁
        long stamp = sl.tryOptimisticRead();
        //将两个字段读入本地局部变量
        //尽量完成所有的读取操作
        //将读取到的数据存入局部变量
        double currentX = x, currentY = y;  
        //验证发出乐观读锁尝试后
        //到这段时间是否有写锁获取
        if (!sl.validate(stamp)) { 
            //如果没有，我们再次获得一个读悲观锁
            stamp = sl.readLock();  
            //将两个字段刷新到本地局部变量
            try {
                currentX = x; 
                currentY = y; 
            } finally {
                sl.unlockRead(stamp);
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }

    //悲观读
    void moveIfAtOrigin(double newX, double newY) { 
        //直接获取读锁
        long stamp = sl.readLock();
        try {
            //循环，检查当前状态是否符合
            //是否在原点
            while (x == 0.0 && y == 0.0) { 
                //当前点在原点
                //可以move
                //将读锁转为写锁
                long ws = sl.tryConvertToWriteLock(stamp);
                //确认转为写锁是否成功
                //如果成功 替换票据
                //并执行move
                if (ws != 0L) { 
                    stamp = ws; 
                    x = newX; 
                    y = newY; 
                    break;
                }
                //如果不能成功转换为写锁 
                else { 
                	//显式释放读锁
                	//显式直接进行写锁 然后再通过循环再试
                    sl.unlockRead(stamp); 
                    stamp = sl.writeLock();  
                }
            }
        } finally {
        	//释放读锁或写锁
            sl.unlock(stamp); 
        }
    }
}
```

## 实现原理

其实乐观锁只是一种思想，它假设每次拿数据的时候其他线程不会去修改，也就是假设了一种低碰撞的场景，在这种场景下不会对资源加锁。那么万一在真正处理拿到的数据进行计算时，其他线程修改了这些数据呢？即使是乐观的锁，也一定要保证计算的准确性。所以在真正运用这些数据计算的时候会再次 **验证**。对这段时间到底有没有线程修改作**验证**。

可以看到，本质上就是CAS的操作。

```java
//尝试乐观读，没有加任何锁
//返回一个状态stamp
public long tryOptimisticRead() {
    long s;
    return (((s = state) & WBIT) == 0L) ? (s & SBITS) : 0L;
}
//验证stamp和state是否相等
//不相等说明这段时间有写锁进入
//所以是不安全的，调用者应该尝试重试或者升级锁
public boolean validate(long stamp) {
    U.loadFence();
    return (stamp & SBITS) == (state & SBITS);
}
```

# 乐观与悲观

乐观锁：适用于多读的情形，提高吞吐量。是一种不直接加锁的策略。都是基于CAS的操作实现。例如：Atomic类，StampedLock中的乐观读。

悲观锁：ReentrantLock直接锁，synchronized关键字的直接使用，其实悲观锁的实现。常用于写操作比较多，冲突比较多的情形。