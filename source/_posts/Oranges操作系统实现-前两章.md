---
title: Oranges操作系统实现-前两章
toc: true
date: 2021-03-29 14:01:09
tags:
categories:
---

前两章主要是一些准备工作，扇区引导，环境配置

<!--more-->

## 引导扇区

开机过程：

计算机电源被打开时，先进行加电自检（**P**ower-**O**n **S**elf-**T**est；POST），这是BIOS的一个功能，针对硬件如CPU、主板、存储器进行检查，速度很快。

BIOS将磁盘的第一个扇区512字节（引导程序）载入内存，放到内存07c00处。

寻找启动盘，比如从软盘启动，就检查软盘的0面0磁道1扇区，如果发现以`0xAA55`结束，BIOS就认为这是一个引导扇区

引导扇区除了以0xAA55（这是一个结束标志）结束，还应该包含512字节的执行码（**510吧？**）

一旦BIOS发现引导扇区，就会将这512字节的内容装入到内存地址07c00h处，然后头转到这里，将控制权交给这段引导代码，计算机的控制就从BIOS转到了操作系统。

```assembly
	org	07c00h			; 告诉编译器程序加载到7c00处
	mov	ax, cs
	mov	ds, ax
	mov	es, ax
	call	DispStr			; 调用显示字符串例程
	jmp	$			; 无限循环 $表示当前行被汇编之后的地址
DispStr:
	mov	ax, BootMessage ;NASM中，没有[]的被认为是地址，访问标签中的内容使用[]
	mov	bp, ax			; ES:BP = 字符串地址
	mov	cx, 16			; CX = 串长度
	mov	ax, 01301h		; AH = 13,  AL = 01h
	mov	bx, 000ch		; 页号为0(BH = 0) 黑底红字(BL = 0Ch,高亮)
	mov	dl, 0
	int	10h			; 10h 号中断
	ret
BootMessage:		db	"Hello, OS world!"
times 	510-($-$$)	db	0	; 填充剩下的空间，使生成的二进制代码恰好为512字节，$$ 表示一个section开始处被汇编后的地址，$-$$表示本行距离程序开始处的相对距离
dw 	0xaa55				; 结束标志
```

用NASM编译后，得到一个512字节的bin文件，用工具写入到第一个扇区上。

## 环境搭建

### Bochs

ubuntu20.04安装bochs，

装bochs-2.6.9

```shell
$tar vxzf bochs-2.6.9.tar.gz

$cd bochs-2.6.9

$./configure --enable-debugger --with-sdl --enable-disasm  --enable-readline LIBS='-lX11'    

$make

$sudo make  install
```



