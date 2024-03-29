---
title: ubuntu配置
toc: true
date: 2020-12-28 15:24:33
tags: 环境配置
categories:
- 环境配置
---

wsl、ubuntu、vm配置相关

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

## 挂载vmware共享文件夹

默认是挂载到`/mnt/hdfs`下，想修改挂载到`home/username`下



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

![image-20210517205327302](./ubuntu配置/image-20210517205327302.png)

注意13行，判断home目录下bashrc文件是否存在，存在则读入，**也就是说login shell环境下，最终读入的配置文件是`~/.bashrc`**，**也就是说自己的偏好设置写入`~/.bashrc`即可。**

#### non-login shell配置文件

只会读取`~/.bashrc`这一个文件

### 相关命令

`source` 读入环境配置文件，比如刚修改了.bashrc文件，source以下就可以生效了 



## 终端shell配置

环境配置

```shell
sudo apt install tmux
sudo apt install zsh
```

wsl设置走clash代理

```shell
export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*');
export https_proxy="http://$hostip:7890";
export http_proxy="http://$hostip:7890";
export all_proxy="socks5://$hostip:7891";
```

安装ohmyzsh

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装.tmux

```shell
cd ~
git clone https://github.com/gpakosz/.tmux.git
ln -s -f .tmux/.tmux.conf
cp .tmux/.tmux.conf.local .
```

安装三个zsh插件

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions

sudo apt install autojump
```

然后在`.zshrc`中配置

```
plugins=(.. zsh-syntax-highlighting zsh-autosuggestions autojump)
[[ -s ~/.autojump/etc/profile.d/autojump.zsh ]] && . ~/.autojump/etc/profile.d/autojump.zsh
autoload -U compinit && compinit -u
```

默认shell修改

`.tmux.conf.local`中加入

```shell
set -g default-shell /bin/zsh
```

注意要让tmux配置生效，直接在shell里source是不行的

要进入tmux，然后ctrl+b，输入`:source ~/.tmux.conf`

安装p10k主题

```shell
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

然后修改.zshrc

```shell
ZSH_THEME="powerlevel10k/powerlevel10k"
tmux #末尾添加，开启shell默认进入tmux
```

最后 `source .zshrc`，会自动配置p10k

## 中文linux命令手册

`pip install how`

## conda

防止conda污染原本的环境，一定一定一定要记住，不要把conda加到环境变量里。

如果之后我们要启动conda，我们可以用如下命令：

source <path_to_your_conda_installation>/bin/activate  # 默认进入base
source <path_to_your_conda_installation>/bin/activate <your_env_name>  # 进入指定环境

## nvim

https://space.bilibili.com/26319956/video?tid=0&page=1&keyword=&order=pubdate

