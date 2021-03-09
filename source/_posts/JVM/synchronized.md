---
title: Synchronized
date: 2021-03-09 20:38:00
author: yancyan
categories: 虚拟机
tags:
- JAVA
- JVM
---

## Synchronized 介绍
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

## Synchronized 锁优化

传统的锁依赖系统的同步函数，在Linux上使用mutex互斥量，最底层实现依赖futex，这些同步函数都涉及到用户态和内核态的切换，进程上下文切换，成本较高。对于加了synchronized关键字但运行时并没有多线程竞争，或两个线程接近于交替执行的情况，使用传统锁机制无疑效率是会比较低的。在JDK 1.6之前,synchronized只有传统的锁机制，因此给开发者留下了synchronized关键字相比于其他同步机制性能不好的印象。

在JDK1.6引入了两种新型锁机制：偏向锁和轻量级锁，它们的引入是为了解决在没有多线程竞争或基本没有竞争的场景下因使用传统锁机制带来的性能开销问题。

在看这几种锁机制的实现前，我们先来了解下对象头，它是实现多种锁机制的基础。
### 对象头

Hotspot虚拟机中的对象头分为两部分，第一部分存储对象自身的运行数据，如哈希码，GC分代年龄等，长度在32和64位虚拟机中分别占32和64个比特，官方称位`Mark Work`。另一部分存储指向方法区对象类型数据的指针，如果是数组对象还会额外存储数组对象长度，由于对象头信息是与对象自身数据无关的额外成本，考虑到Java虚拟机的空间使用效率，Mark Work被设计成一个非固定的动态数据结构。在32位系统上各状态的格式如下：
![对象头](/images/object_header.png)

### 轻量级锁
轻量级锁JDK6加入，并不是为了替代重量级锁，设计初衷是为了在没有多线程竞争的情况下，减少传统重量级锁使用操作系统互斥量产生的性能损耗

#### 加锁：
在代码即将进入同步块时，判断同步对象是否被锁住（锁标志位"01"）,虚拟机首先在栈帧中创建锁记录”Lock Record“空间，用于存储锁对象的”MarkWord“拷贝（Displaced Mark Work），然后使用CAS操作把**锁对象的MarkWork**更新为指向**LockRecord**的指针。更新成功，代表该线程持有了锁，，且对象`MarkWord`的锁标志位变成“00”，表示此对象出于轻量级锁状态。更新失败，表示有线程竞争该锁，虚拟机先检查对象`MarkWord`是否指向当前线程栈帧，不是当前线程，判定为出现多线程竞争锁的情况，那轻量级锁不再有效，需要膨胀为重量级锁，锁标志位变为“10”，此时`MarkWork `存储指向重量级锁的指针，等待锁的线程进入阻塞状态。

#### 解锁
解锁过程同样使用CAS，如果锁对象的`MarkWork`依然指向线程的`LockRecord`, 使用CAS将锁对象当前的`MarkWork`和线程中复制的`Displaced Mark Work`做替换，如果成功，同步结束，失败则说明有其他线程尝试获取过该锁，释放锁的同时唤醒被挂起的线程。

**轻量级锁提升性能的场景为，绝大部分锁，同步周期内都是不存在竞争的，如果存在竞争，除了互斥量开销，还额外发生CAS操作，因此在高并发竞争下，轻量级锁反而比传统锁更慢。**

### 偏向锁

偏向锁JDK6加入，目的是为了消除数据在无竞争情况下的同步原语，进一步提升性能，如果说轻量级锁是在无竞争情况下使用CAS操作消除同步使用的互斥量，那偏向锁就是在无竞争的情况下去掉同步语义。可以通过`-XX:-UseBiasedLocking`禁止偏向锁优化

#### 加锁：
如果虚拟机启用了偏向锁（-XX:+UseBiasedLocking,JKD6以后虚拟机的默认值）。
锁对象第一次被线程获取时，虚拟机会把对象头中的标识位设置为”01“，偏向模式设置”1“，进入偏向模式，同时使用CAS操作把获取到锁的线程ID记录到锁对象的`MarkWork`中，CAS成功，持有偏向锁的线程以后每次进入同步块，虚拟机都不再进行任何同步操作，

#### 解锁
一旦有另外一个线程去尝试获取这个锁，偏向模式马上结束，根据对象释放出于被锁定状态决定是否撤销偏向锁（偏向模式设置”0“），撤销以后标志位恢复到未锁定（标志位”01“）或轻量级锁定（标志位”00“）状态。

**偏向锁需要保持线程ID，HashCode怎么存放呢？**
在Java中一个对象如果计算过hashCode，就应该保持这个值不变，因此当一个对象计算过哈希码后就再也无法进入偏向锁状态，而一个对象如果正出于偏向锁状态，又收到需要计算一致性哈希码的需求，它的偏向锁会立即撤销并且锁会膨胀为重量级锁。在重量级锁的实现中，对象头指向了重量级锁的位置，说明重量级锁ObjectMontor类里有可以记录非加锁状态（标志位”01“）的`MarkWork`信息。

### 锁消除和锁粗化
除了对同步原语的锁机制优化，还提供了锁粒度自动优化，帮忙优化锁的滥用。
#### 消除
即时编译器运行时，对代码要求同步，但是检测到不可能存在共享数据竞争的锁进行消除。
其主要判定依据源于逃逸分析的数据支持，一段代码中，堆上的数据不会逃逸出去被其他线程访问到，就可以把他们都当作栈上数据对待，同步自然不需要存在。除了优化开发人员的同步语义还有许多同步并不是开发人员加入。例如每个JDK5以后的字符串拼接自动转到使用`StringBuffer.append()`，它的每个方法都有一个同步块，锁就是SB对象，
```java
public String concatString(String a, String b, String c){
    StirngBuffer sb = new StringBuffer();
    sb.append(a);
    sb.append(b);
    sb.append(c);
    return sb.toString();
}
```
经过逃逸分析，sb对象作用域限制在方法内，可以进行锁消除优化，即可忽略同步措施。

#### 粗化

代码编写过程中总要求尽量缩小同步块的大小，但是如果一系列的连续操作都对同一个对象返回加锁解锁，甚至加解锁出现在循环内，也会导致不必要的损耗，如消除中举例代码，就可以把加锁同步范围扩展（粗化）到整个操作系列外部，第一个append之前到最后一个append之后。实现仅简化。