---
title: 内存屏障
date: 2021-03-13 11:38:00
author: yancyan
categories: Java
tags:
- JVM
---

## 介绍
内存屏障，就是让CPU处理单元中的内存状态对其他处理单元可见的技术，内存屏障提供了两个功能，首先通过确保从另一个处理核来看屏障两边的所以指令都是正确的程序顺序，保证程序顺序的外部可见性；其次可以实现内存数据的可见性，确保内存数据会同步到CPU缓存子系统。

## Store Barrier
Store屏障，是x86的”sfence“指令，强制所有在store屏障指令之前的store指令，都在该store屏障指令执行之前被执行，并把store缓冲区的数据都刷到CPU缓存。这会使得程序状态对其它CPU可见，这样其它CPU可以根据需要介入。


## Load Barrier
Load屏障，是x86上的”ifence“指令，强制所有在load屏障指令之后的load指令，都在该load屏障指令执行之后被执行，并且一直等到load缓冲区被该CPU读完才能执行之后的load指令。这使得从其它CPU暴露出来的程序状态对该CPU可见，这之后CPU可以进行后续处理。

## Full Barrier
Full屏障，是x86上的”mfence“指令，复合了load和save屏障的功能。

## Java内存模型
Java内存模型中volatile变量在写操作之后会插入一个store屏障，在读操作之前会插入一个load屏障。一个类的final字段会在初始化后插入一个store屏障，来确保final字段在构造函数初始化完成并可被使用时可见。

## 原子指令和Software Locks
原子指令，如x86上的”lock …” 指令是一个Full Barrier，执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU。Software Locks通常使用了内存屏障或原子指令来实现变量可见性和保持程序顺序。
