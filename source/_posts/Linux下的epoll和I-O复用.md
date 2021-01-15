---
title: Linux下的epoll和I/O复用
toc: true
date: 2021-01-11 17:00:58
tags:
categories:
---

学libco过程中遇到了epoll和I/O复用相关概念。

<!--more-->

## I/O多路复用

早期，为了实现一个服务器支持多个客户端的连接，使用fork/thread一个进程/线程的方式去接受并处理请求。

随着网络连接增多，fork/thread模型就不行了。所以有了IO多路复用模型。一个连接到来后，遍历所有的已注册的文件描述符，找到需要处理的文件描述符，也就是select和poll。

由于遍历引起的巨大开销O(n)，在原有的基础上进一步优化，有了epoll的方法。

## epoll

> `epoll`是linux内核的可扩展I/O事件通知机制。

使用一个文件描述符管理多个描述符。

### 程序接口

```c++
int epoll_create(int size);
```

在内核中创建epoll实例，并返回一个epoll文件描述符。size是要监听的文件描述符数量，大于size的话内核会自动扩容。

```c++
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

向epfd对应的内核epoll实例添加、修改或删除对fd事件event的监听。

op 为 `EPOLL_CTL_ADD`, `EPOLL_CTL_MOD`, `EPOLL_CTL_DEL` 分别对应的是添加新的事件，修改文件描述符上监听的事件类型，从实例上删除一个事件。

如果 event 的 events 属性设置了 `EPOLLET` flag，那么监听该事件的方式是边缘触发。

```c++
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

当 timeout 为 0 时，epoll_wait 立即返回。

 timeout 为 -1 时，epoll_wait 会阻塞直到已注册的事件变为就绪。

当 timeout 为一正整数时，epoll 会阻塞直到计时 timeout 毫秒或已注册的事件变为就绪。（因为内核调度延迟，阻塞会略微超过 timeout 毫秒）。

### epoll的实现

#### mmap

`mmap`将用户空间的一块地址和内核空间的一块地址同时映射到相同的一块物理内存地址，这块物理内存对内核和用户均可见，内核可以直接看到epoll监听的句柄。

#### 红黑树

存储epoll监听的套接字。存储到mmap出来的内存。添加或者删除一个套接字（`epoll_ctl`）时，都在红黑树上去处理。

通过`epoll_ctl` 添加进来的事件都会放在红黑树的节点内。

#### 双向链表

先看epoll_wait做了什么：

* 调用ep_poll，当双向链表中无就绪fd，挂起当前进程，直到链表不为空。
* 当文件fd状态改变，不可读/不可写变为可读/可写，对应fd上的回调函数ep_poll_callback会被调用
* ep_poll_callback将相应fd对应的epitem加入链表，导致链表不为空，进程被唤醒，epoll_wait继续执行。

* ep_event_transfer函数将rdlist中的epitem拷贝到txlist,rdlist清空
* ep_send_event，扫描txlist中的每个epitem，调用关联fd对应的poll方法，从而获得fd上较新的events，封装在epoll_event从epoll_wait返回。