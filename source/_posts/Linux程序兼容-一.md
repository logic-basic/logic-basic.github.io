---
title: Linux程序兼容.一
date: 2018-08-28 19:42:31
tags: [Linux, gcc , glibc]
---

## 前言

之前写了一个Linux的程序，本以为把大部分的库都使用静态链接，就可以在所以Linux桌面环境跑了。
但是今天试着装了一个ubuntu12.04。结果就没运行起来(*/ω＼*).

报告了如下错误：
```
./main: /lib/x86_64-linux-gnu/libm.so.6: version `GLIBC_2.27' not found (required by ./main)
```

编译程序的gcc版本为`gcc-8.1`。而目标机器ubuntu12.04上的gcc版本是`gcc-4.6`。

为什么不使用`-static`把所有的库都静态链接进去。因为我们的库使用了dlopen。
dlopen的实现必须依赖于动态库。如果把dlopen也强行静态链接进去。程序运行崩溃了。

那同样也可以使用旧版本gcc进行编译。这样就可以在旧版本的机器上运行了。
但是我们的库太多了，总共有几十个库了吧。全部换旧版本编译代价太大了。
并且，切旧版本gcc。总觉得很不优雅。

所以，我要寻找，用高版本gcc编译，在低版本环境下运行的方法。

## 初步分析

在这里，用`main`来表示我所编译的程序。
首先，可以先使用命令`ldd main`来看看，我的程序使用了哪些动态库。

```
linux-vdso.so.1 =>  (0x00007fffcdff9000)
libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f8e67ddb000)
librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007f8e67bd3000)
libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f8e679b4000)
libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f8e676ae000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f8e672e9000)
/lib64/ld-linux-x86-64.so.2 (0x000055a80c05d000)
```

根据指令的内容，我们发现就只有这几库进行了动态链接。每个库的作用如下：

`libdl.so` 是用来做动态库打开，比如C语言中的`dlopen`就是在这个库中。
`librt.so` rt是realtime的缩写。如果用到和时间相关的方法就需要使用这个库。
`libpthread.so` pthread很著名了，线程库。
`libm.so` 字母m是math的缩写。所以这个库就是数学运算库了。
`libc.so` libc库是最基本的库。是C语言的实现。

其中，`libc.so`是最底层的库。其他的库都会链接`libc.so`。
一下是ldd这些库的结果。

```
ldd /lib/x86_64-linux-gnu/libpthread.so.0

linux-vdso.so.1 =>  (0x00007ffc2c7a0000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f03d4f10000)
/lib64/ld-linux-x86-64.so.2 (0x000056251832d000)
```

```
ldd /lib/x86_64-linux-gnu/librt.so.1

linux-vdso.so.1 =>  (0x00007fff32ff2000)
libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f2989ce7000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f2989922000)
/lib64/ld-linux-x86-64.so.2 (0x0000558bfe6ea000)
```

```
ldd /lib/x86_64-linux-gnu/libdl.so.2

linux-vdso.so.1 =>  (0x00007ffd5d94f000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f7707aa0000)
/lib64/ld-linux-x86-64.so.2 (0x000055c18d802000)
```

```
ldd /lib/x86_64-linux-gnu/libm.so.6

linux-vdso.so.1 =>  (0x00007ffd5d94f000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f7707aa0000)
/lib64/ld-linux-x86-64.so.2 (0x000055c18d802000)
```

上面的这些库，在大多数的Linux桌面发行版都是自带的。并且我们的程序运行报错是`version `GLIBC_2.27' not found`.

所以根据此推断，程序之所以跑不起来，是因为`libc.so`版本不兼容导致。
因为编译器的机器`libc.so`的版本很高。而运行的机器`libc.so`的版本较低。所以不能运行程序。

## 解决思路

首先，最先想到的是能不能把新版的`libc.so`放到和程序的同一文件夹下。
让程序在本地文件夹去找动态库去链接。

但是，这样做是不行的：
1. 可执行程序无法自动搜索当前目录的下动态库。必须手动指定`LD_LIBRARY_PATH`变量 
2. 如果手动指定了本地的libc库。可能会导致系统中其他程序不能正常运行. 
 
---

那么，从GLIBC出发，我们可以看看这个两个libc.so有什么不同
我们可以使用strings命令来查看版本。

首先查看旧版本：

```
strings /lib/x86_64-linux-gnu/libc.so.6 | grep GLIBC

GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_2.13
GLIBC_2.14
GLIBC_2.15
GLIBC_PRIVATE
GNU C Library (Ubuntu EGLIBC 2.19-0ubuntu6.9) stable release version 2.19, by Roland McGrath et al.
```

然后再看看新版本：

```
strings /usr/lib/libc.so.6 | grep GLIBC_

GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_2.13
GLIBC_2.14
GLIBC_2.15
GLIBC_2.16
GLIBC_2.17
GLIBC_2.18
GLIBC_2.22
GLIBC_2.23
GLIBC_2.24
GLIBC_2.25
GLIBC_2.26
GLIBC_2.27
GLIBC_PRIVATE
```

经过对比我们发现，新版的libc.so多了很多的GLIBC的版本。
从GLIBC_2.16到GLIBC_2.27。这些都是旧版所没有的。
就是新版libc所更新的内容。

结合我们程序的报错`not found GLIBC_2.27`。那就说明，我们程序使用了最新的libc的功能。
而旧版本中又没有，所以就报告了如上的错误。

知道了我们的程序使用是使用了新libc.so里的功能。但是具体都有哪些版本呢，我们可以通过readelf命令来看看，程序都使用了哪些版本的GLIBC

```
readelf -s main | grep -oP "GLIBC_[\d\.]*" | sort | uniq

GLIBC_2.10
GLIBC_2.11
GLIBC_2.14
GLIBC_2.15
GLIBC_2.2.5
GLIBC_2.27
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
```

在这里，我们看到所有这个程序用到GLIBC的版本，十分幸运的是，除了2.27。其他的在旧版本中都有。
也就是说，如果我们可以去掉这个GLIBC_2.27的依赖，程序就可以运行。

那么，我们再来看看有哪些符号方法使用了GLIBC_2.27:

```
readelf -s maxhub | grep GLIBC_2.27

328: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND powf@GLIBC_2.27 (22)
380: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND logf@GLIBC_2.27 (22)
```

我们发现，只有两个符号使用了GLIBC_2.27，分别是`powf`和`logf`。
其中`powf` 是继续x的y次幂，`logf`是进行log运算。
这两个符号均属于math库中。刚好也验证了程序运行的报错，是从libm.so中报出的。

## 处理问题

现在已经知道了问题的所在，由于math使用了更新更高效的算法。导致我们的程序无法运行在旧版本的机器上。
难点在于如何处理这个问题，替换`libm.so`？这个方法同样不靠谱。

在查阅了一些资料后，在网上找到了解决办法。既然现在这两个符号使用的新版本的实现。
那能不能让它回退，使用旧版本的实现呢。毕竟新版本的so里有之前所有的版本。而版本就不行。

在这里，我们需要使用内联汇编，来替换我们所要使用的版本。
举个例子，具体实现如下：

```
#include <math.h>

__asm__(".symver logf,logf@GLIBC_2.2.5");
__asm__(".symver powf,powf@GLIBC_2.2.5");

int main(int argc, char**)
{
  return powf(argc, 2.0f) * logf(argc);
}
```

需要说明的是，`__asm__` 这两句要写在符号使用的前面。
更详细的说明，可以访问：https://stackoverflow.com/questions/51716288/fedora-28-glibc-2-27-libm-so-6-logf-and-powf-c/51726222

在我们的程序的源码中并没有使用到这两个符号。所以需要去看每个库的源码，有没有使用powf和logf的。
在linux系统中，可以使用ack命令进行查看，十分方便。

最后，在SDL2的源码中找到了这两个符号，在源码中添加内联汇编后，重新编译库。之后再重新编译程序。
就可以在低版本的机器上运行啦。


## 总结

* libc是linux中最基本的库。几乎所有的库和程序都需要使用到它。
* libc是有很多版本的。我们可以通过命令来查看本机所支持的版本和程序中所用到的版本。
* 静态库的编译会把源代码进行汇编处理。所以内联汇编必须写在库的源代码中。



