---
title: SYNCHRONIZED
date: 2018-10-05 16:38:00
author: yancyan
categories: 虚拟机
tags:
- java
- jvm
---

## synchronized 介绍
Java提供了两种实现同步的基础语义：synchronized方法和synchronized块
```java
public class SyncTest {
    public void syncBlock(){
        synchronized (this){
            System.out.println("hello block");
        }
    }
    public synchronized void syncMethod(){
        System.out.println("hello method");
    }
}
```
使用 `javap -v` 查看class文件和字节码信息
```text
public void syncBlock();
        descriptor: ()V
        flags: ACC_PUBLIC
        Code:
        stack=2, locals=3, args_size=1
        0: aload_0
        1: dup
        2: astore_1
        3: monitorenter        // monitorenter指令进入同步块
        4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        7: ldc           #3                  // String hello block
        9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_1
        13: monitorexit        // monitorexit指令退出同步块
        14: goto          22
        17: astore_2
        18: aload_1
        19: monitorexit        // monitorexit指令退出同步块
        20: aload_2
        21: athrow
        22: return
        Exception table:
        from    to  target type
        4    14    17   any
        17    20    17   any


public synchronized void syncMethod();
        descriptor: ()V
        flags: ACC_PUBLIC, ACC_SYNCHRONIZED      //添加了ACC_SYNCHRONIZED标记
        Code:
        stack=2, locals=1, args_size=1
        0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        3: ldc           #5                  // String hello method
        5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        8: return

```

对于synchronized关键字而言，javac在编译时，会生成对应的monitorenter和monitorexit指令分别对应synchronized同步块的进入和退出，有两个monitorexit指令的原因是：为了保证抛异常的情况下也能释放锁，所以javac为同步代码块添加了一个隐式的try-finally，在finally中会调用monitorexit命令释放锁。

对于synchronized方法而言，javac为其生成了一个ACCSYNCHRONIZED关键字，在JVM进行方法调用时，发现调用的方法被ACCSYNCHRONIZED修饰，则会先尝试获得锁。

## 锁的几种形式

传统的锁依赖系统的同步函数，在Linux上使用mutex互斥量，最底层实现依赖futex，这些同步函数都涉及到用户态和内核态的切换，进程上下文切换，成本较高。对于加了synchronized关键字但运行时并没有多线程竞争，或两个线程接近于交替执行的情况，使用传统锁机制无疑效率是会比较低的。在JDK 1.6之前,synchronized只有传统的锁机制，因此给开发者留下了synchronized关键字相比于其他同步机制性能不好的印象。

在JDK 1.6引入了两种新型锁机制：偏向锁和轻量级锁，它们的引入是为了解决在没有多线程竞争或基本没有竞争的场景下因使用传统锁机制带来的性能开销问题。

在看这几种锁机制的实现前，我们先来了解下对象头，它是实现多种锁机制的基础。
### 对象头

因为在Java中任意对象都可以用作锁，因此必定要有一个映射关系，存储该对象以及其对应的锁信息（比如当前哪个线程持有锁，哪些线程在等待）。一种很直观的方法是，用一个全局map，来存储这个映射关系，但这样会有一些问题：需要对map做线程安全保障，不同的synchronized之间会相互影响，性能差；另外当同步对象较多时，该map可能会占用比较多的内存。

所以最好的办法是将这个映射关系存储在对象头中，因为对象头本身也有一些hashcode、GC相关的数据，所以如果能将锁信息与这些信息共存在对象头中就好了。

在JVM中，对象在内存中除了本身的数据外还会有个对象头，对于普通对象而言，其对象头中有两类信息：mark word和类型指针。另外对于数组而言还会有一份记录数组长度的数据。

类型指针是指向该对象所属类对象的指针，mark word用于存储对象的HashCode、GC分代年龄、锁状态等信息。在32位系统上mark word长度为32字节，64位系统上长度为64字节。为了能在有限的空间里存储下更多的数据，其存储格式是不固定的，在32位系统上各状态的格式如下：
![对象头](/images/object_header.png)



