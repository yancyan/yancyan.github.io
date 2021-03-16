---
title: 线程池
date: 2021-03-15 20:38:00
author: yancyan
categories: Java
tags:
- JAVA
- JUC
---

## 介绍
线程池的工作主要是控制运行的线程的数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大线程的数量，超出数量的线程排队等候，等其他线程执行完毕，再从队列中取出任务来执行。
 

## 作用
- 降低资源消耗。通过重复利用已经创建的线程降低线程创建和销毁的消耗。
- 提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
- 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可统一分配、调优和监控。

## JUC
JDK1.5开始，Java提供了Executor框架来创建不同的线程池。该框架中用到Executor、Executors，ExecutorService、ThreadPoolExecutor这几个类,

![](/images/threadpool-juc.png)

1. Executor提供接口将任务提交和执行过程解耦，并用Runnable表示任务,Executor基于”生产者-消费者”模式，提交任务的操作相当于生产者，执行任务的则相当于消费者。
```java
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```
2. ExecutorService 扩展了Executor接口，添加了一些生命周期管理的方法,

```java
public interface ExecutorService extends Executor {
    void shutdown();

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit);
        
    <T> Future<T> submit(Callable<T> task);
    
    <T> Future<T> submit(Runnable task, T result);
    
    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
    ....
}
```

## 线程池核心构造

```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```
- corePoolSize： 线程池中的常驻核心线程数，默认情况下，线程池中没有任何线程（线程数为0），等有任务到来才创建线程去执行任务。当线程池中的线程数达到corePoolSize后，就会把到达的任务放到缓存队列中。没有任务执行时，线程池大小不一定时corePoolSize，allowCoreThreadTimeOut.
```java
/**
 * Timeout in nanoseconds for idle threads waiting for work.
 * Threads use this timeout when there are more than corePoolSize
 * present or if allowCoreThreadTimeOut. Otherwise they wait
 * forever for new work.
 */
private volatile long keepAliveTime;

/**
 * If false (default), core threads stay alive even when idle.
 * If true, core threads use keepAliveTime to time out waiting
 * for work.
 */
private volatile boolean allowCoreThreadTimeOut;
```

- maximumPoolSize: 线程池中允许的最大线程数，线程池能够容纳的最大线程数，此值必须大于等于1。

- keepAliveTime
空闲线程的存活时间。默认情况下，只有当线程池中的线程数大于corePoolSize时，这个参数才会起作用。当线程数大于corePoolSize时，当空闲时间达到keepAliveTime的值时，多余的空闲线程会被销毁直到只剩下corePoolSize。

- unit: keepAliveTime时间单位
- workQueue：任务队列，用来缓存已提交未执行的任务
- threadFactory：生成线程池中工作线程的线程工厂
- handler: 超出线程范围maximumPoolSize和workQueue队列容量时所使用的拒绝策略。 线程池类库自带4种拒绝策略。
    - AbortPolicy-终止策略。直接抛出一个RejectedExecutionException，也是JDK默认的拒绝策略。
    - CallerRunsPolicy-调用者运行策略。将任务回退到调用者（让调用者执行）降低任务量。
    - DiscardOldestPolicy-抛弃最旧的策略。移除队列中最先进入的任务，并且尝试执行任务。
    - DiscardPolicy-抛弃策略。直接丢弃任务。
## 工作原理
![](/images/thread_pool_process.png)

工作流程：
1. 在创建线程池后，等待提交过来的任务请求。
2. 当调用execute() 方法添加一个请求任务时，线程池会进行一系列判断：
    1. 如果正在运行的线程数小于corePoolSize ，那么马上创建线程执行这个任务。
    2. 如果正在运行的线程数大于或等于corePoolSize，那么将这个任务放入队列。
    3. 如果队列满且正在运行的线程数小于maximumPoolSize，那么还是要创建非核心线程立即执行这个任务。
    4. 如果队列满且正在运行的线程数大于或等于maximumPoolSize，那么线程池会启动饱和拒绝策略来执行。
3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。
4. 当一个线程无事可做超过一定的单位（unit）时间（keepAliveTime）时，线程池会判断：
    1. 如果当前运行的线程数大于corePoolSize，那么这个线程就会被销毁。
    2. 如果线程池的全部任务完成，线程池将会收缩到corePoolSize大小。allowCoreThreadTimeOut：true ，核心线程等待超时关闭。
    
 