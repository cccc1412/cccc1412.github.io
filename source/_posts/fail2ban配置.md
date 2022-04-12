---
title: fail2ban配置
toc: true
date: 2021-11-19 15:54:41
tags:
categories:
---

记录一次服务器被黑的经历，，

<!--more-->

看到有奇怪的进程在跑，sudo切换到其用户下，看history。。

![image-20211119162948803](fail2ban配置/image-20211119162948803.png)

## fail2ban配置

```shell
sudo apt install fail2ban
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local


```

