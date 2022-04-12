---
title: Linux内核之块IO层
toc: true
date: 2022-02-21 11:11:38
tags:
categories:
---

![image-20220221113129721](Linux内核之块IO层/image-20220221113129721.png)

<!--more-->

## Linux IO体系

### VFS层

屏蔽了不同文件系统的差异，想上层提供统一的文件读写访问接口

### FS层

具体的文件系统层
