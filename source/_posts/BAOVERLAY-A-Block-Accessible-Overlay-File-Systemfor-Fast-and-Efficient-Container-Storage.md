---
title: >-
  BAOVERLAY: A Block-Accessible Overlay File System for Fast and Efficient
  Container Storage
toc: true
date: 2022-05-11 20:53:15
tags:
categories:
---

<!--more-->

## BAOVerlay设计

BAOVerlay将overlay文件看做一系列块，通过块，baoverlay实现了非阻塞的写实拷贝机制，每次要写入时，并不是完整的文件从下层到上层进行拷贝，而是需要更新的块从下层到上层异步复制。最后，baoverlay实现了一种新的文件格式B-Cow，用来紧凑存储overlay文件用来节省存储空间。分配存储空间被推迟，知道实际实际更新了overlay文件的块。

## 实现块访问

在POSIX兼容的文件系统中，文件或目录与inode数据结构关联，inode存储了文件的元信息，例如文件的所有权、访问模式和磁盘中的数据位置，该文件的操作。overlay文件系统为每个

overlay2的open函数会创建一个关联到overlay文件的 inode，并填充所需要的信息。其中一个主要的步骤就是找出关联的underlaying文件属于哪一层，从而根据backing fs的inode 的fetch操作，然后将其放入overlay inode的一个字段中。
