---
title: ptmalloc分析
toc: true
date: 2021-10-08 10:40:10
tags:
categories:
---

内存分配器位于用户程序和内核之间，向操作系统申请内存，然后返回给用户程序。

分配器会先向系统批发一块大于用户请求的内存，然后用某种算法进行管理，零售给用户的每次内存请求。

<!--more-->

## 内存管理数据结构

### 主分配区和非主分配区

> 设置分配区是为了支持多线程

每个进程只有一个主分配区，可能有多个非主分配区。分配区的数量一旦增加，就不会再减少了。

主分配区和非主分配区通过环形链表进行管理。每个分配区使用互斥锁进行访问互斥。



## 内存分配

ptmalloc响应用户内存分配要求的步骤为：



## 内存回收

具体的释放方式是看chunk所处的位置和chunk的大小

* 获得分配区的锁
* 
