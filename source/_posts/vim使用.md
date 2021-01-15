---
title: vim使用
toc: true
date: 2020-11-03 11:22:55
tags:
categories:
---

<!--more-->

## Vim基础

### 编辑模式

vim有五种操作模式

* 正常模式：在文件中移动光标
* 插入模式：插入文本
* 替换模式：替换文本
* 可视化模式（块模式）：选中文本
* 命令模式：执行命令

从正常模式进入其他模式：

* i 进入插入模式insert
* R（大写）替换模式
* v进入可视模式view  或者ctrl+v
* :进入命令模式

esc键使用会很多，可以考虑键位重定义

### 缓存、标签页、窗口

vim维护的一系列打开的文件，称为缓存

### 命令行

- :q 退出
- :w 保存（写）
- :wq 保存退出
- :e {文件名} 打开要编辑的文件
- :ls 显示打开的缓存
- :help {标题} 打开帮助文档

## 移动

* 翻页：ctrl+u往上 ，ctrl+d往下
* 词移动：w下一个词，b上一个词，e下一个词尾
* 行：0移动到行初，^到行第一个非空格字符，$移动到行尾
* 屏幕：H屏幕首行、M屏幕中间、L屏幕底部
* 查找：f{字符}
* 搜索：/{正则表达式}

## 编辑

* O在上面插入行、o在下面插入行
* d{移动命令}，删除，dw删除词，d$删除到行尾
* c{移动命令}，改变
* x删除字符
* s替换字符
* 可视化+操作 选中可以d或c
* u撤销
* y复制
* p粘贴

### 计数

计数可以执行指定操作若干次，如：

- `3w` 向前移动三个词
- `5j` 向下移动5行
- `7dw` 删除7个词

### 修饰语

？？

### 自定义vim

来自：https://missing-semester-cn.github.io/2020/editors/

```shell
" Comments in Vimscript start with a `"`.

" If you open this file in Vim, it'll be syntax highlighted for you.

" Vim is based on Vi. Setting `nocompatible` switches from the default
" Vi-compatibility mode and enables useful Vim functionality. This
" configuration option turns out not to be necessary for the file named
" '~/.vimrc', because Vim automatically enters nocompatible mode if that file
" is present. But we're including it here just in case this config file is
" loaded some other way (e.g. saved as `foo`, and then Vim started with
" `vim -u foo`).
set nocompatible

" Turn on syntax highlighting.
syntax on

" Disable the default Vim startup message.
set shortmess+=I

" Show line numbers.
set number

" This enables relative line numbering mode. With both number and
" relativenumber enabled, the current line shows the true line number, while
" all other lines (above and below) are numbered relative to the current line.
" This is useful because you can tell, at a glance, what count is needed to
" jump up or down to a particular line, by {count}k to go up or {count}j to go
" down.
set relativenumber

" Always show the status line at the bottom, even if you only have one window open.
set laststatus=2

" The backspace key has slightly unintuitive behavior by default. For example,
" by default, you can't backspace before the insertion point set with 'i'.
" This configuration makes backspace behave more reasonably, in that you can
" backspace over anything.
set backspace=indent,eol,start

" By default, Vim doesn't let you hide a buffer (i.e. have a buffer that isn't
" shown in any window) that has unsaved changes. This is to prevent you from "
" forgetting about unsaved changes and then quitting e.g. via `:qa!`. We find
" hidden buffers helpful enough to disable this protection. See `:help hidden`
" for more information on this.
set hidden

" This setting makes search case-insensitive when all characters in the string
" being searched are lowercase. However, the search becomes case-sensitive if
" it contains any capital letters. This makes searching more convenient.
set ignorecase
set smartcase

" Enable searching as you type, rather than waiting till you press enter.
set incsearch

" Unbind some useless/annoying default key bindings.
nmap Q <Nop> " 'Q' in normal mode enters Ex mode. You almost never want this.

" Disable audible bell because it's annoying.
set noerrorbells visualbell t_vb=

" Enable mouse support. You should avoid relying on this too much, but it can
" sometimes be convenient.
set mouse+=a

" Try to prevent bad habits like using the arrow keys for movement. This is
" not the only possible bad habit. For example, holding down the h/j/k/l keys
" for movement, rather than using more efficient movement commands, is also a
" bad habit. The former is enforceable through a .vimrc, while we don't know
" how to prevent the latter.
" Do this in normal mode...
nnoremap <Left>  :echoe "Use h"<CR>
nnoremap <Right> :echoe "Use l"<CR>
nnoremap <Up>    :echoe "Use k"<CR>
nnoremap <Down>  :echoe "Use j"<CR>
" ...and in insert mode
inoremap <Left>  <ESC>:echoe "Use h"<CR>
inoremap <Right> <ESC>:echoe "Use l"<CR>
inoremap <Up>    <ESC>:echoe "Use k"<CR>
inoremap <Down>  <ESC>:echoe "Use j"<CR>
```

### 扩展vim

只需要创建一个 `~/.vim/pack/vendor/start/` 的文件夹， 然后把插件放到这里 （比如通过 `git clone`）。

常用插件：

* [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim): 模糊文件查找
* [ack.vim](https://github.com/mileszs/ack.vim): 代码搜索
* [nerdtree](https://github.com/scrooloose/nerdtree): 文件浏览器
* [vim-easymotion](https://github.com/easymotion/vim-easymotion): 魔术操作



## 打开目录树

```shell
#vim命令模式下
:Explore #当前窗口下打开
:Vexplore #竖直分割窗口打开
:Sexplore #水平分割窗口打开

#命令打开文件夹
root@localhost：vim /etc
```

