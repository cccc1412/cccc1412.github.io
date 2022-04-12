---
title: Efficient IO with io_uring
toc: true
date: 2022-03-22 15:16:08
tags:
categories:
---

本文旨在介绍最新的Linux  IO接口，io_uring，并将其与现有的进行比较。我们将讨论它存在的原因、内部工作原理以及用户接口。本文不会详细介绍具体的命令，这些在man手册中。相反，本文将介绍io_uring以及其工作原理。 

<!--more-->

## Introduction

在Linux中有很多基于文件的IO方式。最古老的是read和write系统调用。后来有了pread和pwrite的版本，可以传入偏移量参数。在后来又有了preadv和pwritev，这是基于前者的向量版本，进一步扩展了API以及可修改的标志位。这几个系统调用有一个共同点，他们都是同步接口。这就意味这这些调用在数据准备就绪时才能返回。在一些场景下，需要异步接口，POSIX有aio_read和aio_write来满足，但是这两个的实现和性能一般。

Linux的原生异步IO接口，简称aio，有许多局限性：

* 最大的限制是**只支持O_DIRECT**。由于这个限制，native aio在大部分场景下都不适用，对于buffered IO，aio接口以同步的方式运行。
* 即使满足了异步的所有约束，有时候还不是异步的。**IO提交可能会以多种方式阻塞**，如果需要元数据来执行IO，提交将会阻塞。对于存储设备，只有固定数量的请求slots可用，如果这些slots当前被占用，提交将会被阻塞直到有一个可用。
* API本身设计不好。每次IO提交需要拷贝64+8字节，每次完成拷贝32字节，共104字节的内存拷贝。对IO来说，希望的是零拷贝。

这意味着Linux缺乏一个能满足异步IO的接口，在内核可以更高效实现这一点的情况下，应用程序没有理由要创建私有IO线程池来获得良好的异步IO。

## Improving the status quo

一开始的工作集中在改进aio接口上，在被放弃之前，这项工作取得了相当大的进展。出发点是，改善原有接口比提供新的接口简单，工作量小。

现有aio接口由三个系统调用组成：

* io_setuo：设置aio上下文的系统调用。
* io_submit：提交io。
* io_getevents：用于获取或等待io完成的系统调用。

由于这些系统调用需要改变行为，要添加新的系统调用来传递这些信息，最终使得代码复杂度和可维护性都不是很好，API的理解和使用也变得更复杂。

所以有必要从零开始设计一个全新的东西。

## 新的接口设计目标

* 易用，不易误用。
* 可扩展，新的接口不只是用于块IO，对于网络或非块存储也能用。

* 功能丰富。要涵盖应用程序所需要的大部分功能（比如IO线程池），减少应用程序开发的工作量。
* 效率高。存储IO大多是基于块的，因此一个IO大小至少为512b或4kb，协议请求甚至没有携带数据负载，新接口要求对每个请求的开销都高效。
* 可伸缩性。虽然效率和低延迟很重要，但是在峰值也要尽可能提供最佳的新能。

## io_uring

aio为了处理IO尽量了多个单独内存拷贝，也就是存在内存副本，损害了效率和可扩展性，这是要避免的。

由于要尽量避免复制，内核和应用程序必须友好的共享定义IO本身和完成事件的结构，要实现这一点，需要让共享数据的coordination也驻留在应用程序和内核之间的共享内存。要实现这一点，就要考虑这内核和应用程序的同步管理方式。

应用程序不能在不用系统调用的情况下与内核共享锁，系统调用肯定会降低与内核通信的速率，这违背了设计目标中的效率需求。满足我们需求的一种数据结构是ringbuffer，单生产者单消费者的场景。通过共享环形缓冲区，我们可以消除应用程序和内核之间共享锁的需要，而且巧妙的避开了memory和ordering和barriers。

异步接口有两个基本操作：提交请求的行为和完成请求的事件。对于提交IO，应用程序时生产者，内核是消费者。对于完成事件刚好相反，内核生产，用户消费。因此我们需要一对环buffer来在应用程序和内核之间提供有效的通信通道。这个ringbuffer的通信通道就是io_uring的核心，也就是后面的提交队列（SQ）和完成队列（CQ），这是新接口的基础。

### data structure

有了通信通道，下面要定义用于描述请求和完成事件的数据结构。

#### cqe完成队列事件

> Completion Queue Event

对于完成队列事件，简称cqe：

```c++
struct io_uring_cqe { 
    __u64    user_data; 
    __s32    res; 
    __u32    flags; 
};
```

* user_data用户数据字段取自初始提交请求，可以包含应用程序识别请求所需要的任何信息。一个常见的用例是将其作为原始请求的指针，内核不会触及这个字段，只是在提交和完成过程中简单搬运。
* res保存请求的结果，将其视为系统调用的返回值。对于正常的读写操作，类似于读或写的返回值。对于成功的操作，他将包含传输的字节数，如果发生故障，返回负数。

#### sqe请求队列条目

> Submission Queue Entry

请求队列(sqe)定义：

```c++
struct io_uring_sqe { 
    __u8    opcode; 
    __u8    flags; 
    __u16    ioprio; 
    __s32    fd; 
    __u64    off; 
    __u64    addr; 
    __u32    len; 
    union { 
        __kernel_rwf_t    rw_flags; 
        __u32        fsync_flags; 
        __u16        poll_events; 
        __u32        sync_range_flags; 
        __u32        msg_flags;             
    }; 
    __u64    user_data; 
    union { 
        __u16    buf_index; 
        __u64    __pad2[3]; 
    }; 
};
```

* opcode：操作码，用于描述特定请求的操作码，比如矢量读取IORING_OP_READV
* flag：
* ioprio是对此请求的优先级，对于正常的读写操作，遵循ioprio_set中的定义，
* fd是与请求关联的文件描述符
* off保存操作应该发生传递偏移量
* addr：如果操纵码描述了传输数据的操作，addr包含操作应执行IO的地址。如果操作码时某种向量化读写操作，那么将指向struct iovec数组的指针，跟preadv类似。对于非矢量IO传输，addr直接包含地址。
* len是非矢量IO传输的字节计数，或者是addr描述的矢量数，用于矢量IO传输。 

* ...

### communication channel

介绍ring如何工作的。尽管提交和完成是对称的，但是两者之间的索引是不同的。

CQE被组织成一个数组，该数组的内存可以被内核和应用程序可读可写。然而CQE是内核产生的，因此实际上只能由内核修改。

每当内核想cq ring发布新事件时，都会更新tail，当应用程序消费entry时，更新头部。因此，如果head和tail不同，应用程序就知道有多个事件可以消费。head和tail时32位int，可以自然溢出，这样就不用管理ring已满的标志。ring的大小必须是2的n次幂。

要查找一个事件的索引，应用程序必须使用环的大小掩码来屏蔽当前尾部的索引。

```c++
unsigned head; 

head = cqring→head; 
read_barrier(); 
if (head != cqring→tail) { 
    struct io_uring_cqe *cqe; 
    unsigned index; 

    index = head & (cqring→mask); 
    cqe = &cqring→cqes[index]; 
    /* process completed cqe here */ 
    ... 
    /* we've now consumed this entry */ 
    head++; 
} 

cqring→head = head; 
write_barrier();
```

虽然CQ环直接索引CQ的共享数组，但提交端在他们之间有一个间接数组。因此提交端环形缓冲区是该数组的索引，而该数组又包含进入SQE的索引。原因是，一些应用程序可能回在内部数据结构中嵌入请求单元，使他们能灵活的这样做，同时保留在一个操作中提交多个SQE的能力。

sqe的使用和cqe基本是相反的操作：

```c++
struct io_uring_sqe *sqe; 
unsigned tail, index; 

tail = sqring→tail; 
index = tail & (*sqring→ring_mask); 
sqe = &sqring→sqes[index]; 

/* this call fills in the sqe entries for this IO */ 
init_io(sqe); 

/* fill the sqe index into the SQ ring array */ 
sqring→array[index] = index; 
tail++; 

write_barrier(); 
sqring→tail = tail; 
write_barrier();
```

一旦内核使用了sqe



## 参考资料

[异步IO框架io_uring介绍]: https://arthurchiao.art/blog/intro-to-io-uring-zh/

