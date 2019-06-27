---
title: linux下使用dmesg调试程序崩溃
date: 2019-06-22 10:46:46
tags: [Linux, 调试]
---

## dmesg输出

dmesg是linux系统中，内核打印输出.
我们在内核中使用printk. 或者我们在用户空间写/dev/kmsg，也可以输出到dmesg

## 一个例子

我们编写一个简单的程序main.cpp
```
 1
 2 #include <iostream>
 3
 4 void func() {
 5     int *p = nullptr;
 6     *p = 1;
 7 }
 8
 9 int main() {
10     func();
11
12     return 0;
13 }
```

编译之后运行

当我们用户的程序崩溃时，操作系统会在dmesg打印一条信息。比如：

```
[  194.230017] main[727]: segfault at 0 ip 0000001de200e869 sp 00007ffeb667cb20 error 6 in main[1de200e000+1000]
```

其中,
ip为 指令寄存器
sp为 栈顶的偏移地址

我们主要通过ip来判断位置，但是，如果我们运行多次，就会发现，ip的值，每次都不一样：
```
[  840.467566] main[752]: segfault at 0 ip 0000009e40290869 sp 00007ffc615c5720 error 6 in main[9e40290000+1000]
[  841.546995] main[765]: segfault at 0 ip 000000430da30869 sp 00007ffc6fb54990 error 6 in main[430da30000+1000]
[  842.570851] main[777]: segfault at 0 ip 000000fc2df61869 sp 00007fff3d5eee50 error 6 in main[fc2df61000+1000]
```

经过观察，可以发现，每次ip的最后三位都是一样的。同时，我们发现main的后面有一个+1000。
我们通过计算可以发现：
```
0x0000009e40290869 % 0x1000 = 0x869
0x000000430da30869 % 0x1000 = 0x869
0x000000fc2df61869 % 0x1000 = 0x869
```
这个0x869应该就是我们的代码段地址


使用`addr2line -Cfe main 869`
输出：
```
func()
/home/dangjiahe/main.cpp:6
```
至此，我们可以定位到程序具体的崩溃位置，第二行，需要编译时加入-g参数，如果没有-g，只能定位到func()函数。


## 一些思考

1. 为什么偏移是0x1000
猜测：操作系统对于内存最小的操作单位是页，一页在现在常用的操作系统的大小是4096字节。刚好是0x1000
      我们的代码比较简单，代码段的大小还超不过一页，所以偏移就是1页的大小0x1000


2. 如何通过0x869定位具体的位置
通过addr2line命令，可以直接查到。
我们也可以通过`objdump -d main`
以下是部分输出：
```
0000000000000859 <_Z4funcv>:
 859:	55                   	push   %rbp
 85a:	48 89 e5             	mov    %rsp,%rbp
 85d:	48 c7 45 f8 00 00 00 	movq   $0x0,-0x8(%rbp)
 864:	00
 865:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
 869:	c7 00 01 00 00 00    	movl   $0x1,(%rax)
 86f:	90                   	nop
 870:	5d                   	pop    %rbp
 871:	c3                   	retq

0000000000000872 <main>:
 872:	55                   	push   %rbp
 873:	48 89 e5             	mov    %rsp,%rbp
 876:	e8 de ff ff ff       	callq  859 <_Z4funcv>
 87b:	b8 00 00 00 00       	mov    $0x0,%eax
 880:	5d                   	pop    %rbp
 881:	c3                   	retq
```
我们发现，刚好在0x869, movl操作，就是我们给nullptr指针写操作。同时函数是`_Z4funcv`, 转换过来就是func()
如果没有-g参数，我们就只能定位到这个func函数，还有对于函数入口的偏移。-g 会把行号信息加入进去


