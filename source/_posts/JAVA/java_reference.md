---
title: Java Reference
date: 2021-03-08 21:38:00
author: yancyan
categories: Java
tags:
- JAVA
---
 
## 引用
Java中4种引用的强度由高到低依次为：强引用 -> 软引用 -> 弱引用 -> 虚引用

| 引用类型  | 被垃圾回收时机   | 用途 | 生存时间 |
| :--- | :--- | --- | ---|
| 强引用       | 从来不会        | 对象的一般状态| JVM停止运行时终止  |
| 软引用       | 当内存不足前    | 对象缓存| 内存不足时终止  |
| 弱引用       | GC一看到立刻回收 | 对象缓存| 垃圾回收后终止  |
| 虚引用       | 随时随刻        | 跟踪对象的垃圾回收| 垃圾回收后终止  |

Java 引用类
- FinalReference，一种保底策略，因为GC只能管理自动内存资源而无法管理其它资源（如堆外内存、file handle、socket等），这些需要使用方手动对资源进行管理
- SoftReference，软引用，只有在堆内存不足时，垃圾回收器会回收对应引用。所以比较适合用来实现不是特别重要的缓存
- WeakReference，弱引用，每次垃圾回收都会回收其引用，一般在需要控制内存但又又想要尽量用到内存的场景下使用
- PhantomReference，虚引用，对引用无影响，只用于获取对象被回收的通知。和软引用以及弱引用不同的是幻影引用指向的对象没有其他强引用、软引用指向时不会自动被GC清理。

说明
- 强引用没有对应的类型表示，也就是说强引用是普遍存在的，如Object object = new Object();
- 软引用、弱引用和虚引用都是java.lang.ref.Reference的直接子类。
- 直到JDK11为止，只存在四种引用，这些引用是由JVM创建，因此直接继承java.lang.ref.Reference创建自定义的引用类型是无效的，但是可以直接继承已经存在的引用类型，如java.lang.ref.Cleaner就是继承自java.lang.ref.PhantomReference。
- 特殊的java.lang.ref.Reference的子类java.lang.ref.FinalReference和Object#finalize()有关，java.lang.ref.Finalizer是java.lang.ref.FinalReference子类。

### 软引用 (SoftReference)
```java
public class SoftReferenceMain {

	public static void main(String[] args) throws Exception {
		ReferenceQueue<SoftReferenceObject> queue = new ReferenceQueue<>();
		SoftReferenceObject object = new SoftReferenceObject();
		SoftReference<SoftReferenceObject> reference = new SoftReference<>(object, queue);
		object = null;
		System.gc();
		Thread.sleep(500);
		System.out.println(reference.get());
	}

	private static class SoftReferenceObject {

		int[] array = new int[120_000];

		@Override
		public String toString() {
			return "SoftReferenceObject";
		}
	}
}
// 运行后输出结果
null
```

### 弱引用 (WeakReference)

ThreadLocal
```java
static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
}
```
简单示例
```java
public class WeakReferenceMain {

	public static void main(String[] args) throws Exception {
		ReferenceQueue<WeakReferenceObject> queue = new ReferenceQueue<>();
		WeakReferenceObject object = new WeakReferenceObject();
		System.out.println(object);
		WeakReference<WeakReferenceObject> reference = new WeakReference<>(object, queue);
		object = null;
		System.gc();
		Thread.sleep(500);
		System.out.println(reference.get());
	}

	private static class WeakReferenceObject {

		@Override
		public String toString() {
			return "WeakReferenceObject";
		}
	}
}
// 运行后输出结果
WeakReferenceObject
null
```

### 虚引用(PhantomReference)
一个对象是否关联到虚引用，完全不会影响该对象的生命周期，也无法通过虚引用来获取一个对象的实例。 为对象设置一个虚引用的唯一目的是：能在此对象被垃圾收集器回收的时候收到一个系统通知。

PhantomReference有两个比较常用的子类是java.lang.ref.Cleaner和jdk.internal.ref.Cleaner，其中前者提供的功能是开发者用于在引用对象回收的时候触发一个动作(java.lang.ref.Cleaner$Cleanable)，后者用于DirectByteBuffer对象回收的时候对于堆外内存的回收，可以翻看前面描述java.lang.ref.Reference#processPendingReferences()源码的时候，ReferenceHandler线程会对pending链表中的jdk.internal.ref.Cleaner类型引用对象调用其clean()方法。
```java
public class PhantomReferenceTest {
    public static void main(String[] args) {
        ReferenceQueue<String> referenceQueue = new ReferenceQueue<>();
        PhantomReference<String> phantomReference = new PhantomReference<>("hello",referenceQueue);
        System.out.println(phantomReference.get());
    }
} 
```

## Reference && ReferenceQueue 

### ReferenceQueue
ReferenceQueue队列是一个单向链表，ReferenceQueue里面只有一个head 成员变量持有队列的队头。后进先出的队列，其实是个就是个栈!

ReferenceQueue和 Reference 类都是 jdk1.2 的时候出的，所以也就不可能继承jdk1.5出来的Queue接口

### Reference
这是Reference对象的抽象base类，这个类定义了所有 reference 对象的通用方法，因为 reference 对象跟GC垃圾收集器密切相关.

源码说明
```text
/* The state of a Reference object is characterized by two attributes.  It
* may be either "active", "pending", or "inactive".  It may also be
* either "registered", "enqueued", "dequeued", or "unregistered".
*
*   Active: Subject to special treatment by the garbage collector.  Some
*   time after the collector detects that the reachability of the
*   referent has changed to the appropriate state, the collector
*   "notifies" the reference, changing the state to either "pending" or
*   "inactive".
*   referent != null; discovered = null, or in GC discovered list.
*
*   Pending: An element of the pending-Reference list, waiting to be
*   processed by the ReferenceHandler thread.  The pending-Reference
*   list is linked through the discovered fields of references in the
*   list.
*   referent = null; discovered = next element in pending-Reference list.
*
*   Inactive: Neither Active nor Pending.
*   referent = null.
*
*   Registered: Associated with a queue when created, and not yet added
*   to the queue.
*   queue = the associated queue.
*
*   Enqueued: Added to the associated queue, and not yet removed.
*   queue = ReferenceQueue.ENQUEUE; next = next entry in list, or this to
*   indicate end of list.
*
*   Dequeued: Added to the associated queue and then removed.
*   queue = ReferenceQueue.NULL; next = this.
*
*   Unregistered: Not associated with a queue when created.
*   queue = ReferenceQueue.NULL.
*
* The collector only needs to examine the referent field and the
* discovered field to determine whether a (non-FinalReference) Reference
* object needs special treatment.  If the referent is non-null and not
* known to be live, then it may need to be discovered for possible later
* notification.  But if the discovered field is non-null, then it has
* already been discovered.
*
* FinalReference (which exists to support finalization) differs from
* other references, because a FinalReference is not cleared when
* notified.  The referent being null or not cannot be used to distinguish
* between the active state and pending or inactive states.  However,
* FinalReferences do not support enqueue().  Instead, the next field of a
* FinalReference object is set to "this" when it is added to the
* pending-Reference list.  The use of "this" as the value of next in the
* enqueued and dequeued states maintains the non-active state.  An
* additional check that the next field is null is required to determine
* that a FinalReference object is active.
*
* Initial states:
*   [active/registered]
*   [active/unregistered] [1]
*
* Transitions:
*                            clear
*   [active/registered]     ------->   [inactive/registered]
*          |                                 |
*          |                                 | enqueue [2]
*          | GC              enqueue [2]     |
*          |                -----------------|
*          |                                 |
*          v                                 |
*   [pending/registered]    ---              v
*          |                   | ReferenceHandler
*          | enqueue [2]       |--->   [inactive/enqueued]
*          v                   |             |
*   [pending/enqueued]      ---              |
*          |                                 | poll/remove
*          | poll/remove                     |
*          |                                 |
*          v            ReferenceHandler     v
*   [pending/dequeued]      ------>    [inactive/dequeued]
*
*
*                           clear/enqueue/GC [3]
*   [active/unregistered]   ------
*          |                      |
*          | GC                   |
*          |                      |--> [inactive/unregistered]
*          v                      |
*   [pending/unregistered]  ------
*                           ReferenceHandler
*
* Terminal states:
*   [inactive/dequeued]
*   [inactive/unregistered]
*
* Unreachable states (because enqueue also clears):
*   [active/enqeued]
*   [active/dequeued]
*
* [1] Unregistered is not permitted for FinalReferences.
*
* [2] These transitions are not possible for FinalReferences, making
* [pending/enqueued] and [pending/dequeued] unreachable, and
* [inactive/registered] terminal.
*
* [3] The garbage collector may directly transition a Reference
* from [active/unregistered] to [inactive/unregistered],
* bypassing the pending-Reference list.
*/


一个引用对象可以同时存在两种状态：
- 第一组状态："active", "pending", or "inactive"
- 第二组状态："registered", "enqueued", "dequeued", or "unregistered"

Active：
当前引用实例处于Active状态，会收到垃圾收集器的特殊处理。在垃圾收集器检测到referent的可达性已更改为适当状态之后的某个时间，垃圾收集器会"通知"当前引用实例改变其状态为"pending"或者"inactive"。此时的判断条件是：referent != null; discovered = null或者实例位于GC的discovered列表中。

Pending：
当前的引用实例是pending-Reference列表的一个元素，等待被ReferenceHandler线程处理。pending-Reference列表通过应用实例的discovered字段进行关联。此时的判断条件是：referent = null; discovered = pending-Reference列表中的下一个元素

Inactive：
当前的引用实例处于非Active和非Pending状态。此时的判断条件是：referent = null (同时discovered = null)

Registered：
当前的引用实例创建的时候关联到一个引用队列实例，但是引用实例暂未加入到队列中。此时的判断条件是：queue = 传入的ReferenceQueue实例

Enqueued：
当前的引用实例已经添加到和它关联的引用队列中但是尚未移除(remove)，也就是调用了ReferenceQueue.enqueued()后的Reference实例就会处于这个状态。此时的判断条件是：queue = ReferenceQueue.ENQUEUE; next = 引用列表中的下一个引用实例，或者如果当前引用实例是引用列表中的最后一个元素，则它会进入Inactive状态

Dequeued：
当前的引用实例曾经添加到和它关联的引用队列中并且已经移除(remove)。此时的判断条件是：queue = ReferenceQueue.NULL; next = 当前的引用实例

Unregistered：
当前的引用实例不存在关联的引用队列，也就是创建引用实例的时候传入的queue为null。此时的判断条件是：queue = ReferenceQueue.NULL
```


![](/images/java_reference_main.png)

ReferenceQueue队列的作用就是Reference引用的对象被回收时，Reference对象能否进入到pending队列，最终由ReferenceHander线程处理后，Reference就被放到了这个队列里面（Cleaner对象除外），然后我们就可以在这个ReferenceQueue里拿到reference,执行我们自己的操作，所以这个队列起到一个对象被回收时通知的作用； 

如果不带ReferenceQueue的话,要想知道Reference持有的对象是否被回收，就只有不断地轮训reference对象,通过判断get是否为null(phantomReference对象不能这样做)。这两种方法均有相应的使用场景,取决于实际的应用.如weakHashMap中就选择去查询queue的数据,来判定是否有对象将被回收.而ThreadLocalMap,则采用判断get()是否为null来作处理; 

对于带ReferenceQueue参数的构造方法，如果传入的队列为null，那么就会给成员变量queue赋值为ReferenceQueue.NULL队列,这个NULL是ReferenceQueue对象的一个继承了ReferenceQueue的内部类，它重写了入队方法enqueue，这个方法只有一个操作，直接返回 false，也就是这个对列不会存取任何数据,它起到状态标识的作用； 

***Cleaner对象是没有Enqueue状态的，它经过HandReference处理时执行其clean方法清理，然后就直接进入了inactive状态***

## Cleaner
sun.misc.Cleaner是JDK内部提供的用来释放非堆内存资源的API。JVM只会帮我们自动释放堆内存资源，但是堆外内存无能为力，该类提供了回调机制，通过这个类能方便的释放系统的其他资源。Cleaner 继承了 PhantomReference，是一个虚幻引用。



