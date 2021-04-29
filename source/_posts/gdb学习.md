---
title: gdb学习
toc: true
date: 2021-01-09 12:32:34
tags:
categories:
---

gdb笔记

<!--more-->

### GDB功能

> gdb有四个主要功能：

* 启动程序，指定任何可能影响其行为的内容
* 打断点，让程序在特定条件下停下来
* 当程序停止时，检查发生了什么
* 改变程序变量

### 启动gdb

> 命令行输入:

* gdb program

* gdb program core
* gdb program 1234 或者 gdb -p 1234 ，调试一个正在运行的进程

### 常用命令

* `break [file:]function`   在function中设置断点
* `run [arglist]`  使用arglist启动程序

* `print expr` 显示表达式的值
* `c`  继续运行程序
* `next` 执行下一行代码，step over function
* `step` 执行下一行代码，step into function
* `help [name]` 显示name的帮助，name是命令
* `quit` 退出gdb

### 命令选项

* `-help` / `-h` 列出所有选项
* `-symbols=file` / `-s file` 从file中读符号表

* `-write` 开启写入到可执行与核心文件？？
* `-exec=file` / `-e file` 