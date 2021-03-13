---
title: Reactor模型
date: 2021-03-11 21:11:00
author: yancyan
categories: 事务
tags:
- Seata
---

## 介绍
在高性能的I/O设计中，有两个著名的模型：Reactor模型和Proactor模型，其中Reactor模型用于同步I/O，而Proactor模型运用于异步I/O操作。

Reactor模型基于事件驱动，特别适合海量IO事件
Rector模型中定义三种角色
- Reactor：监听和分配事件，将连接建立就绪、读就绪、写就绪等事件分派给Handler。
- Acceptor：处理客户端连接，并分配请求到处理器链
- Handler：绑定到事件，完成Channel读入，完成业务处理，完成结果写出Channel，可用资源池管理

## 单Reactor单线程模型
Reactor线程负责多路分离套接字，accept新连接并分派到handler。Redis使用单Reactor单进程模的模型



消息处理流程
1. Reactor对象通过select监控连接事件，收到事件后通过dispath转发；
2. 