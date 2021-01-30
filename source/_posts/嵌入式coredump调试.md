---
title: 嵌入式coredump调试
date: 2020-01-08 17:32:49
tags: [coredump, 嵌入式]
---

嵌入式程序奔溃，一般不好定位。不能直接gdb调试。用dmesg结合addr2line呢，有时候根据地址也查不到具体的位置，因为可能最后在so中崩溃。

为此，如果空间够大，可以开启coredump来生成coredump文件。然后使用gdb来进行调试


##### 开启生成coredump

```
echo "/mnt/sdcard/core-%p-%e.$(date "+%Y%m%d_%H%M%S")" > /proc/sys/kernel/core_pattern
ulimit -c unlimited 
```

这样，程序崩溃后,coredump就会生成在生成在/mnt/sdcard目录,如果要放在其他目录,改前面的目录即可


##### 复现问题

程序奔溃，生成coredump


##### 运行gdb

使用gdb + coredump文件的方式运行gdb

这个gdb，必须是交叉编译器附带的gdb，否则可能不识别coredump文件


##### 查看缺少的动态库

刚加载coredump文件, 还缺少很多so文件，比如libc.so, libc++so等等, 缺少这些so，我们的堆栈信息是显示不了的。

使用`info sharedlibrary` 来查看缺少哪些库

##### 添加缺少的动态库

使用`set solib-search-path` 来添加so的路径

这个路径一般在交叉编译器相关目录，如果有多个目录，则使用;来分隔

添加完目录之后，既可使用bt来查看崩溃时的堆栈

