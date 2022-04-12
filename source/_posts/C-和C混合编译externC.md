---
title: C++和C混合编译externC
toc: true
date: 2021-11-04 20:58:14
tags:
categories:
---

.c文件和.cpp文件在混合编译时遇到了undefined reference to XX的问题。

<!--more-->

## 出现undefined reference to 的原因

同一个函数名，C++和C编译后的符号不同，造成链接器找不到XX函数的定义了。

程序目录如下：

![image-20211104211001937](\C-和C混合编译externC\image-20211104211001937.png)

test.cpp中需要用到my_malloc.c中的函数，如果在test.cpp中不用extern "C"修饰my_malloc.h，test.o的符号表如下：

![image-20211104211254599](\C-和C混合编译externC\image-20211104211254599.png)

可以看到，使用C++规则进行编译后，符号表中的函数名添加了前缀和后缀，这跟C++的**重载实现(符号名中加入形参信息)**和**调用约定（如fastcall，stdcall）**有关。

这也说明了为什么不能根据返回值类型的不同实现函数的重载，因为符号表中函数名的生产规则没考虑返回值，即使返回类型不同也会出现重定义。

## 解决

使用extern "C"包裹my_malloc.h后重新编译test.cpp ：`gcc -c test.cpp`

```c++
//file test.cpp
#include <stdio.h>
#include <stdlib.h>
extern "C"{
#include "my_malloc.h"
}
...
```

得到test.o的符号表如下：

![image-20211104211819832](\C-和C混合编译externC\image-20211104211819832.png)

这样就跟my_malloc.o中的符号保持了一致，链接可以正常完成：

![image-20211104211907736](\C-和C混合编译externC\image-20211104211907736.png)



## 参考

https://zhuanlan.zhihu.com/p/123269132



