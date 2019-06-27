---
title: 使用gdb调试android程序
date: 2019-06-26 10:02:45
tags: [android, gdb, 调试]
---

## 前言

之前有一个jni接口调用java方法，在android程序中出现可卡死的问题。
因为接口调用比较频繁，没法通过log来排查问题。这个问题又需要很长时间来复现，所以必须通过现场调试才行。


## gdbserver

gdbserver 可以attach一个进程。同时可以监听在某一个端口来等待gdb的连接

在android里面，如果是64位程序，则需要使用gdbserver64

```
gdbserver64 0.0.0.0:9999 --attach 10952
```

如上，就是监听在9999端口，attach 进程号为10952的进程


## gdb调试

gdbserver监听之后，就可以使用gdb来调试了


1. `gdb`

2. `target remote 10.10.10.50:9999` 端口前面是android程序所在的ip

之后，gdb会加载so等信息

3. 加载完之后，使用`continue` 即可继续让程序运行

## 保存所有堆栈信息

想保存堆栈信息时，可以按下ctrl+c . 就可以中断程序运行。

设置log文件：
`(gdb) set logging file /tmp/log.txt`

开始采集log
`(gdb) set logging on`

输出所有的线程堆栈
`(gdb) thread apply all bt`

关闭log采集
`(gdb) set logging on`

之后，退出gdb就可以看到log了

## 备注

在gdb加载完进程时，可能会出现:

```
Thread 1 "eenshare.server" received signal SIG33, Real-time event 33.
```
线程会收到33的信号，这个我们可以暂时忽略。直接再按c。继续。直到程序继续运行。不会影响程序运行结果
具体为什么会收到33, 这个可能还需要再继续研究.


