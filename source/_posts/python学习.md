---
title: python学习
toc: true
date: 2019-10-26 13:57:13
tags: python
categories:
- 编程语言
- python
---

资料来自网络。

<!--more-->

## 杂记

* `pass`语句

```python
def sample(n_samples):
    pass
```

 pass 占据一个位置，因为如果定义一个空函数程序会报错。

可以在模块划分是使用，先把要写的函数名写好，用pass占位，之后函数细节再慢慢补。

* `list()`函数

将元组转换为列表，可以去重。一个坑是，转化后不会保持元组中的顺序。

## 字符串

| 操作符 | 描述                                                         | 实例                                 |
| :----- | :----------------------------------------------------------- | :----------------------------------- |
| +      | 字符串连接                                                   | >>>a + b 'HelloPython'               |
| *      | 重复输出字符串                                               | >>>a * 2 'HelloHello'                |
| []     | 通过索引获取字符串中字符                                     | >>>a[1] 'e'                          |
| [ : ]  | 截取字符串中的一部分                                         | >>>a[1:4] 'ell'                      |
| in     | 成员运算符 - 如果字符串中包含给定的字符返回 True             | >>>"H" in a True                     |
| not in | 成员运算符 - 如果字符串中不包含给定的字符返回 True           | >>>"M" not in a True                 |
| r/R    | 原始字符串 - 原始字符串：所有的字符串都是直接按照字面的意思来使用，没有转义特殊或不能打印的字符。 原始字符串除在字符串的第一个引号前加上字母"r"（可以大小写）以外，与普通字符串有着几乎完全相同的语法。 | >>>print r'\n' \n >>> print R'\n' \n |



## OS模块

*   `os.sep`:取代操作系统特定的路径分隔符
*   `os.name`:指示你正在使用的工作平台。比如对于Windows，它是'nt'，而对于Linux/Unix用户，它是'posix'。
*   `os.getcwd`:得到当前工作目录，即当前python脚本工作的目录路径。
*   `os.getenv()`和`os.putenv`:分别用来读取和设置环境变量
*   `os.listdir()`:返回指定目录下的所有文件和目录名
*   `os.remove(file)`:删除一个文件
*   `os.stat（file）`:获得文件属性
*   `os.chmod(file)`:修改文件权限和时间戳
*   `os.mkdir(name):`创建目录
*   `os.rmdir(name)`:删除目录
*   `os.removedirs（r“c：\python”）`:删除多个目录
*   `os.system()`:运行shell命令
*   `os.exit()`:终止当前进程
*   `os.linesep`:给出当前平台的行终止符。例如，Windows使用'\\r\\n'，Linux使用'\\n'而Mac使用'\\r'
*   `os.path.split()`:返回一个路径的目录名和文件名
*   `os.path.isfile()`和`os.path.isdir()`分别检验给出的路径是一个目录还是文件
*   `os.path.existe()`:检验给出的路径是否真的存在
*   `os.listdir(dirname)`:列出dirname下的目录和文件
*   `os.getcwd()`:获得当前工作目录
*   `os.curdir`:返回当前目录（'.'）
*   `os.chdir(dirname)`:改变工作目录到dirname
*   `os.path.isdir(name)`:判断name是不是目录，不是目录就返回false
*   `os.path.isfile(name)`:判断name这个文件是否存在，不存在返回false
*   `os.path.exists(name)`:判断是否存在文件或目录name
*   `os.path.getsize(name)`:或得文件大小，如果name是目录返回0L
*   `os.path.abspath(name)`:获得绝对路径
*   `os.path.isabs()`:判断是否为绝对路径
*   `os.path.normpath(path)`:规范path字符串形式
*   `os.path.split(name)`:分割文件名与目录（事实上，如果你完全使用目录，它也会将最后一个目录作为文件名而分离，同时它不会判断文件或目录是否存在）
*   `os.path.splitext()`:分离文件名和扩展名
*   `os.path.join(path,name)`:连接目录与文件名或目录
*   `os.path.basename(path)`:返回文件名
*   `os.path.dirname(path)`:返回文件路径

## 文件操作

`os.mknod("text.txt")`：创建空文件
`fp = open("text.txt",w)`:直接打开一个文件，如果文件不存在就创建文件

### 关于`open`的模式

w 写方式
a 追加模式打开（从EOF开始，必要时创建新文件）
r+ 以读写模式打开
w+ 以读写模式打开
a+ 以读写模式打开
rb 以二进制读模式打开
wb 以二进制写模式打开 (参见 w )
ab 以二进制追加模式打开 (参见 a )
rb+ 以二进制读写模式打开 (参见 r+ )
wb+ 以二进制读写模式打开 (参见 w+ )
ab+ 以二进制读写模式打开 (参见 a+ )

### 关于文件的函数

```
fp.read([size])
```

size为读取的长度，以byte为单位

```
fp.readline([size])
```

读一行，如果定义了size，有可能返回的只是一行的一部分

```
fp.readlines([size])
```

把文件每一行作为一个list的一个成员，并返回这个list。其实它的内部是通过循环调用readline()来实现的。如果提供size参数，size是表示读取内容的总长，也就是说可能只读到文件的一部分。

```
fp.write(str)
```

把str写到文件中，write()并不会在str后加上一个换行符

```
fp.writelines(seq)
```

把seq的内容全部写到文件中(多行一次性写入)。这个函数也只是忠实地写入，不会在每行后面加上任何东西。

```
fp.close()
```

关闭文件。python会在一个文件不用后自动关闭文件，不过这一功能没有保证，最好还是养成自己关闭的习惯。 如果一个文件在关闭后还对其进行操作会产生ValueError

```
fp.flush()
```

把缓冲区的内容写入硬盘

```
fp.fileno()
```

返回一个长整型的”文件标签“

```
fp.isatty()
```

文件是否是一个终端设备文件（unix系统中的）

```
fp.tell()
```

返回文件操作标记的当前位置，以文件的开头为原点

```
fp.next()
```

返回下一行，并将文件操作标记位移到下一行。把一个file用于for … in file这样的语句时，就是调用next()函数来实现遍历的。

```
fp.seek(offset[,whence])
```

将文件打操作标记移到offset的位置。这个offset一般是相对于文件的开头来计算的，一般为正数。但如果提供了whence参数就不一定了，whence可以为0表示从头开始计算，1表示以当前位置为原点计算。2表示以文件末尾为原点进行计算。需要注意，如果文件以a或a+的模式打开，每次进行写操作时，文件操作标记会自动返回到文件末尾。

```
fp.truncate([size])
```

把文件裁成规定的大小，默认的是裁到当前文件操作标记的位置。如果size比文件的大小还要大，依据系统的不同可能是不改变文件，也可能是用0把文件补到相应的大小，也可能是以一些随机的内容加上去。

## 目录操作

```
os.mkdir("file")
```

创建目录

复制文件:

```
shutil.copyfile("oldfile","newfile")
```

oldfile和newfile都只能是文件

```
shutil.copy("oldfile","newfile")
```

oldfile只能是文件夹，newfile可以是文件，也可以是目标目录

```
shutil.copytree("olddir","newdir")
```

复制文件夹.olddir和newdir都只能是目录，且newdir必须不存在

```
os.rename("oldname","newname")
```

重命名文件（目录）.文件或目录都是使用这条命令

```
shutil.move("oldpos","newpos")
```

移动文件（目录）

```
os.rmdir("dir")
```

只能删除空目录

```
shutil.rmtree("dir")
```

空目录、有内容的目录都可以删

```
os.chdir("path")
```

转换目录，换路径

##  属性操作：hasattr()、getattr()、setattr()函数的使用

* `hasattr(object, name)`
  判断一个对象里面是否有name属性或者name方法，返回BOOL值，有name特性返回True， 否则返回False。需要注意的是name要用括号括起来

  ```python
  >>> class test():
  ...     name="xiaohua"
  ...     def run(self):
  ...             return "HelloWord"
  ...
  >>> t=test()
  >>> hasattr(t, "name") #判断对象有name属性
  True
  >>> hasattr(t, "run")  #判断对象有run方法
  True
  >>>
  ```

* `getattr(object, name[,default])`

  获取对象object的属性或者方法，如果存在打印出来，如果不存在，打印出默认值，默认值可选。
  需要注意的是，如果是返回的对象的方法，返回的是方法的内存地址，如果需要运行这个方法，
  可以在后面添加一对括号。

  ```python
  >>> class test():
  ...     name="xiaohua"
  ...     def run(self):
  ...             return "HelloWord"
  ...
  >>> t=test()
  >>> getattr(t, "name") #获取name属性，存在就打印出来。
  'xiaohua'
  >>> getattr(t, "run")  #获取run方法，存在就打印出方法的内存地址。
  <bound method test.run of <__main__.test instance at 0x0269C878>>
  >>> getattr(t, "run")()  #获取run方法，后面加括号可以将这个方法运行。
  'HelloWord'
  >>> getattr(t, "age")  #获取一个不存在的属性。
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  AttributeError: test instance has no attribute 'age'
  >>> getattr(t, "age","18")  #若属性不存在，返回一个默认值。
  '18'
  >>>
  ```

* setattr(object, name, values)

  给对象的属性赋值，若属性不存在，先创建再赋值。

  ```python
  >>> class test():
  ...     name="xiaohua"
  ...     def run(self):
  ...             return "HelloWord"
  ...
  >>> t=test()
  >>> hasattr(t, "age")   #判断属性是否存在
  False
  >>> setattr(t, "age", "18")   #为属相赋值，并没有返回值
  >>> hasattr(t, "age")    #属性存在了
  True
  >>>
  ```

  一种综合的用法是：判断一个对象的属性是否存在，若不存在就添加该属性。

  ```python
  >>> class test():
  ...     name="xiaohua"
  ...     def run(self):
  ...             return "HelloWord"
  ...
  >>> t=test()
  >>> getattr(t, "age")    #age属性不存在
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  AttributeError: test instance has no attribute 'age'
  >>> getattr(t, "age", setattr(t, "age", "18")) #age属性不存在时，设置该属性
  '18'
  >>> getattr(t, "age")  #可检测设置成功
  '18'
  >>>
  ```

  