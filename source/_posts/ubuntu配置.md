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