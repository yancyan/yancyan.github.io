---
title: Java Lock
date: 2021-03-08 21:38:00
author: yancyan
categories: Java
tags:
- JAVA
- JVM
- LOCK
---

Java中往往按照是否含有某一特性定义锁，这里也先根据特性将锁进行归类
![](/images/java_lock.png)

## 乐观锁 - 悲观锁
乐观锁与悲观锁是一种广义上的概念，体现在对待线程同步的看法。在Java和数据库中都有此概念对应的实际应用。

对于同一个数据的并发操作，悲观锁认为自己在使用数据的时候一定有别的线程来修改数据，在获取数据的时候会先加锁，确保数据不会被别的线程修改。Java中，synchronized关键字和Lock的实现类都是悲观锁。 而乐观锁认为不会有别的线程修改数据，先不加锁，只在更新数据时判断数据有没有被其他线程更新。

**悲观锁适合写操作多的场景，先加锁保证写操作正确**
**乐观锁适合读操作多的场景，不加锁提升并发性**

乐观锁在Java中是通过使用无锁编程来实现，最常采用的是CAS算法，Java原子类中的递增操作就通过CAS自旋实现的。
CAS虽然很高效，但是它也存在三大问题：
1. ABA问题
   ABA问题的解决思路就是在变量前面添加版本号，JDK从1.5开始提供了AtomicStampedReference类来解决ABA问题

2. 循环时间长开销大
   CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。

3. 只能保证一个共享变量的原子操作
   Java从1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。

## 自旋锁 - 适应性自旋锁
阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态转换需要耗费处理器时间。如果同步代码块中的内容过于简单，状态转换消耗的时间有可能比用户代码执行的时间还要长。

自旋要占用CPU，如果锁被占用时间较长，那只会浪费处理器资源，所以自旋必须有一个限度，如果超过了限定次数（默认是10次，可以使用-XX:PreBlockSpin来更改）没有成功获得锁，就应该挂起线程

自旋锁在JDK1.4.2中引入，使用-XX:+UseSpinning来开启。JDK 6中变为默认开启，并且引入了自适应的自旋锁（适应性自旋锁）。

自适应性自旋锁的自旋时间（次数）不再固定，由前一次在同一个锁上的**自旋时间**及**锁的拥有者的状态**来决定。同一个锁对象上，自旋等待杠成功获取锁且拥有锁的线程正在运行，虚拟机认为自旋成功概率较大，可适当延长自旋时间，

在自旋锁中 另有三种常见的锁形式:TicketLock、CLHlock和MCSlock，待后续细看

## 无锁 - 偏向锁 - 轻量级锁 - 重量级锁
这四种是指锁状态，针对`synchronized`, 它是悲观锁，在操作同步资源前先加锁，锁存放于对象头中
以Hotspot为例，对象头包括两部分，MarkWord和类型指针
- Mark Word
  默认存储对象的HashCode，分代年龄和锁标志位信息
- 类型指针 
  指向对象的类元数据，虚拟机通过这个指针来确定这个对象是哪个类的实例

锁膨胀：简单来说就是锁类型从偏向锁到轻量级锁再到重量级锁依次演变的过程。
- 偏向锁：只有一个线程进入临界区；
- 轻量级锁：多个线程交替进入临界区；
- 重量级锁：多个线程同时进入临界区。

## 公平锁 - 非公平锁
公平锁是指多个线程按照申请锁的顺序来获取锁，线程直接进入队列中排队，队列中的第一个线程才能获得锁。优点是等待锁的线程不会饿死，缺点是整体吞吐效率相对非公平锁要低，唤醒阻塞线程的开销比非公平锁大。

非公平锁是多个线程加锁时直接尝试获取锁，获取不到才会到等待队列的队尾等待。但如果此时锁刚好可用，那么这个线程可以无需阻塞直接获取到锁。非公平锁的优点是可以减少唤起线程的开销，整体的吞吐效率高，因为线程有几率不阻塞直接获得锁，CPU不必唤醒所有线程。缺点是处于等待队列中的线程可能会饿死，或者等很久才会获得锁。

以**ReentrantLock**为例，ReentrantLock里面有一个内部类Sync，Sync继承AQS（AbstractQueuedSynchronizer），添加锁和释放锁的大部分操作实际上都是在Sync中实现的。它有公平锁FairSync和非公平锁NonfairSync两个子类。ReentrantLock默认使用非公平锁，也可以通过构造器来显示的指定使用公平锁。

```java 
# FairSync 公平锁同步器
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

```java 
# 非公平锁同步器
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
通过上图中的源代码对比，可以看出公平锁与非公平锁的lock()方法区别就在于公平锁在获取同步状态时多了一个限制条件hasQueuedPredecessors()【主要是判断当前线程是否位于同步队列中的第一个】 ,公平锁就是通过同步队列来实现多个线程按照申请锁的顺序来获取锁，从而实现公平的特性。非公平锁加锁时不考虑排队等待问题，直接尝试获取锁.

## 可重入锁 - 不可重入锁
可重入锁是指在同一线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），Java中ReentrantLock和synchronized都是可重入锁，可重入锁的一个优点是可一定程度避免死锁

## 独享锁 - 共享锁
独享锁也叫排他锁，是指该锁一次只能被一个线程所持有。JDK中的synchronized和JUC中Lock的实现类就是互斥锁。

共享锁是指该锁可被多个线程所持有。如果线程T对数据A加上共享锁后，则其他线程只能对A再加共享锁，不能加排它锁。获得共享锁的线程只能读数据，不能修改数据。

独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享,如`ReentrantReadWriteLock`

```java
    private final ReentrantReadWriteLock.ReadLock readerLock;
    /** Inner class providing writelock */
    private final ReentrantReadWriteLock.WriteLock writerLock;
    /** Performs all synchronization mechanics */
    final Sync sync;
```
AQS中使用`int state`标记有多少个线程持有锁，独享锁通常值为0或则1，共享锁就是持有锁的数量，但是在ReentrantReadWriteLock中有读、写两把锁，所以需要在一个整型变量state上分别描述读锁和写锁的数量（或者也可以叫状态）。于是将state变量“按位切割”切分成了两个部分，高16位表示读锁状态，低16位表示写锁状态（写锁个数）。如下图所示：
```java
// 写锁
protected final boolean tryAcquire(int acquires) {
        Thread current = Thread.currentThread();
        int c = getState(); // 取到当前锁的个数
        int w = exclusiveCount(c); // 取写锁的个数w
        if (c != 0) { // 如果已经有线程持有了锁(c!=0)
        // (Note: if c != 0 and w == 0 then shared count != 0)
        if (w == 0 || current != getExclusiveOwnerThread()) // 如果写线程数（w）为0（换言之存在读锁） 或者持有锁的线程不是当前线程就返回失败
        return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)    // 如果写入锁的数量大于最大数（65535，2的16次方-1）就抛出一个Error。
        throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
        }
        if (writerShouldBlock() || !compareAndSetState(c, c + acquires)) // 如果当且写线程数为0，并且当前线程需要阻塞那么就返回失败；或者如果通过CAS增加写线程数失败也返回失败。
        return false;
        setExclusiveOwnerThread(current); // 如果c=0，w=0或者c>0，w>0（重入），则设置当前线程或锁的拥有者
        return true;
        }
```
- **首先取到当前锁的个数c，然后再通过c来获取写锁的个数w**。因为写锁是低16位，所以取低16位的最大值与当前的c做与运算（ int w = exclusiveCount©; ），高16位和0与运算后是0，剩下的就是低位运算的值，同时也是持有写锁的线程数目。
- 取到写锁线程的数目后，首先判断是否已经有线程持有了锁。如果已经有线程持有了锁(c!=0)，则查看当前写锁线程的数目，如果写线程数不为0（即此时存在读锁）或者持有锁的线程不是当前线程就返回失败
- 如果写锁的数量大于最大数（65535，2的16次方-1）就抛出一个Error
- 如果当且写线程数为0（那么读线程也应该为0，因为上面已经处理c!=0的情况），并且当前线程需要阻塞那么就返回失败；如果通过CAS增加写线程数失败也返回失败。
- 如果c=0,w=0或者c>0,w>0（重入），则设置当前线程或锁的拥有者，返回成功

```java
// 读锁
protected final int tryAcquireShared(int unused) {
        Thread current = Thread.currentThread();
        int c = getState();
        if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;                                   // 如果其他线程已经获取了写锁，则当前线程获取读锁失败，进入等待状态
        int r = sharedCount(c);
        if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        if (r == 0) {
        firstReader = current;
        firstReaderHoldCount = 1;
        } else if (firstReader == current) {
        firstReaderHoldCount++;
        } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
        cachedHoldCounter = rh = readHolds.get();
        else if (rh.count == 0)
        readHolds.set(rh);
        rh.count++;
        }
        return 1;
        }
        return fullTryAcquireShared(current);
        }
```
可以看到在tryAcquireShared(int unused)方法中，如果其他线程已经获取了写锁，则当前线程获取读锁失败，进入等待状态。如果当前线程获取了写锁，则当前线程增加读状态，成功获取读锁。读锁的每次释放（线程安全的，可能有多个读线程同时释放读锁）均减少读状态，减少的值是“1<<16”。所以读写锁才能实现读读的过程共享，而读写、写读、写写的过程互斥。

