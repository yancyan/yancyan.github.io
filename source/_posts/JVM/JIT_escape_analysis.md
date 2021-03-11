---
title: JIT - 逃逸分析
date: 2021-03-11 21:11:00
author: yancyan
categories: 虚拟机
tags:
- JIT
- JVM
- 编译优化
---

## 介绍
逃逸分析，与类型继承关系分析一样，并不是直接优化代码的手段，而是为其他优化措施提供依据的分析技术。

## 基本原理
分析对象的动态作用域，在方法体中被定义的变量可能被外部方法所引用，如作为调用参数传递到其他方法中，这种称为方法逃逸。 甚至可能被外部线程访问到，如赋值给可以在其他线程中访问的实例变量，这种称为线程逃逸。从不逃逸、方法逃逸到线程逃逸，称为对象由高到低的逃逸程度。

## 优化手段
逃逸程度不同，可以有不同的优化手段
### **栈山分配（Stack Allocations）**：
对于不会逃逸出线程之外的对象，可以在栈上分配内存，对象所占的内存空间随栈帧出栈而销毁，也会减轻垃圾回收的压力。栈上分配可以支持方法逃逸，但是不能支持线程逃逸。

### **标量替换（Scalar Replacement）**：
一个数据无法分解成更小的数据来表示，Java虚拟机中的原始数据类型（基本类型和Reference类型）都不能再分解，这些数据就称为标量。如果可以继续分解就称为聚合量（Aggregate），如Java对象。

原理: 如果把一个对象拆散，根据程序访问的情况，将其用到的成员变量恢复为原始类型访问，这个过程就叫标量替换

对象不会被方法外部访问（不会方法逃逸），且可被拆散，那程序执行时不会真正的创建这个对象，改为创建被使用的成员变量来代码，除了可以让对象的成员变量在栈上，还可以为进一步的优化创造条件。


### **同步消除（Synchronization Elimination）**：
如果逃逸分析能够确定不会发生线程逃逸，那这个变量就不存在竞争，可以安全的消除同步措施



> JDK6,Hotspot才开始支持初步的逃逸分析，到现在仍不成熟，主要原因是逃逸分析的成本非常高，如果要完全准确判断一个对象是否逃逸，需要一系列复制的数据流敏感的过程间分析。目前虚拟机采用不那么准确的时间压力相对较小的算法完成分析。
> 
> 由于逃逸分析的成本和准确率，在JDK7才默认开启，-XX: +DoEscapeAnalysis 可调，开启以后可通过-XX:+PrintEscapeAnalysis查看分析结果，有了逃逸分析可通过参数-XX：EliminateAllocations开启标量替换，使用-XX:+EliminateLocks开启同步消除，使用-XX:+PrintEliminateAllocations查看标量替换的结果

## 逃逸分析的流程示例

初始代码
```java
public int test(int x){
    int xx = x + 2;
    Point p = new Point(xx, 42);
    return p.getX();
        }

```

第一步，将Point构造函数和getX（）方法进行内联优化
```java
public int test(int x){
    int xx = x + 2;
    Point p = point_memory_alloc(); // 堆上分配p
        p.x = xx; // 构造方法被内联
        p.y = 42;
    return p.x; // Point::getX 方法被内联
        }

```

第二步，逃逸分析,发现不会方法逃逸，进行标量替换
```java
public int test(int x){
    int xx = x + 2;
    int px = xx;
    int py = 42
    return py;
```

数据量分析，进行无用代码消除
```java
public int test(int x){
    return x + 2;
```
