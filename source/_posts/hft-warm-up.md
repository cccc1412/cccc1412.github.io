---
title: hft warm up
toc: true
date: 2022-04-06 14:52:46
tags:
categories:
---

<!--more-->

### 内存管理

[Linux 内存管理](https://zhuanlan.zhihu.com/p/67059173?utm_source=wechat_session&amp;utm_medium=social&amp;utm_oi=786355128126029824&amp;utm_campaign=shareopn)

[C++内存管理](https://zhuanlan.zhihu.com/p/344377490?utm_source=wechat_session&amp;utm_medium=social&amp;utm_oi=786355128126029824&amp;utm_campaign=shareopn)

### hft低延迟技术

[编程细节](https://www.zhihu.com/question/23185359/answer/936467060?utm_source=wechat_session&amp;utm_medium=social&amp;utm_oi=786355128126029824&amp;utm_content=group3_Answer&amp;utm_campaign=shareopn)

[硬件架构](https://www.zhihu.com/question/21922144/answer/27336856?utm_source=wechat_session&amp;utm_medium=social&amp;utm_oi=786355128126029824&amp;utm_content=group3_Answer&amp;utm_campaign=shareopn)

### 系统性能调优

[Linux低延迟服务系统调优](https://zhuanlan.zhihu.com/p/58669088?utm_source=wechat_session&amp;utm_medium=social&amp;utm_oi=786355128126029824&amp;utm_campaign=shareopn)

[Linux 对CPU core 屏蔽timer interrupt](https://zhuanlan.zhihu.com/p/58669088?utm_source=wechat_session&amp;utm_medium=social&amp;utm_oi=786355128126029824&amp;utm_campaign=shareopn)

[Linux获取ns时间戳](https://zhuanlan.zhihu.com/p/58669088?utm_source=wechat_session&amp;utm_medium=social&amp;utm_oi=786355128126029824&amp;utm_campaign=shareopn)

[RDTSC的坑](http://www.wangkaixuan.tech/?p=901)

[Linux 性能优化](https://zhuanlan.zhihu.com/p/141451255?utm_source=wechat_session&utm_medium=social&s_r=0)

## CPU

[CPU如何读写数据、CPU如何选择线程](https://zhuanlan.zhihu.com/p/276170146?utm_source=wechat_session&utm_medium=social&utm_oi=786355128126029824&utm_campaign=shareopn&s_r=0)

[CPU cache](https://zhuanlan.zhihu.com/p/266592261?utm_source=wechat_session&utm_medium=social&utm_oi=786355128126029824&utm_campaign=shareopn&s_r=0)

### 基本工具

perf、strace、netstat、lsof、vtune

## 理解内存性能









## kungfu

### 易筋经

一个journal由多个page组成，一个page由page header和若干个frame组成。一个frame由一个frame header和data组成。frame是数据的最小写入单元。

一个journal只能有一个写入线程，writer在写入时，每一次都是通过一个院子操作在journal中形成一个frame。

每个写入线程对应了一种特定的应用，如行情接受，交易下单等。

![image-20220418150835834](hft-warm-up/image-20220418150835834.png)

![image-20220418150915067](hft-warm-up/image-20220418150915067.png)

## x86指令相关
https://www.felixcloutier.com/x86/