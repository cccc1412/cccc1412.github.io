---
title: ubuntu配置
toc: true
date: 2020-12-28 15:24:33
tags:
categories:
---

<!--more-->

## VMware tools安装

左上虚拟机—>设置—>添加—>CD/DVD—>D:\VMware\VMware Workstation\linux.iso

进入虚拟机后，将tools压缩文件复制进documents文件夹

```shell
# 进入目录
$ cd Documents
# 解压
$ tar -zxvf VMwareTools-10.2.5-8068393.tar.gz
# 进入到解压之后的目录
$ cd vmware-tools-distrib
# 执行 .pl 文件
$ ./vmware-install.pl
```

后面yes回车

## bashrc和profile

### shell和bash

shell是壳程序，能对操作系统和应用程序进行调用的接口程序。这是一个大类

bash是shell其中的一种，历史上还有sh、csh等等，现在linux默认使用的都是bash。

### login shell 和 non-login shell

登录式shell 是以tty中的login、ssh守护进程或其他类似方式派生出来。

非登录式式不需要重复登录的，比如登录Linux后，启动终端terminal,并没有要求重新输入账号和密码。

### 不同的shell类型读取的配置文件不一样

#### login shell配置文件

ctrl+h可显示隐藏.文件

按顺序读两个

先读`/etc/profile`，这个是系统的配置文件，对所有用户生效。一般不要改这个

然后读`~/.profile` 个人用户配置文件，其实个人配置文件可能会有三个`.bash_profile`、`.bash_login`、`.profile`，依次读这三个文件，直到一个存在停止读取，这样是为了应其他shell转换过来的用户的不同习惯。

ubuntu20.04下是`.profile`

![image-20210517205327302](D:\code\myBlog\Blog\source\_posts\ubuntu配置\image-20210517205327302.png)

注意13行，判断home目录下bashrc文件是否存在，存在则读入，**也就是说login shell环境下，最终读入的配置文件是`~/.bashrc`**，**也就是说自己的偏好设置写入`~/.bashrc`即可。**

#### non-login shell配置文件

只会读取`~/.bashrc`这一个文件

### 相关命令

`source` 读入环境配置文件，比如刚修改了.bashrc文件，source以下就可以生效了 

