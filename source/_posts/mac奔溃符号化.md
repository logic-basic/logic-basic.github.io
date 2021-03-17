---
title: mac奔溃符号化
date: 2021-03-17 10:36:01
tags: [macos]
---

## 前言

我们的程序，在出错的时候会产生崩溃。为了找出崩溃的原因，我们需要查看崩溃的堆栈。来分析崩溃时的调用线程和顺序。

在windows平台。崩溃文件是.dump文件。在分析dump文件时，需要结合pdb文件来进行地址到符号的转换。

在mac平台，崩溃文件是.crash文件。如果需要对地址进行转换，则需要.dSYM文件来进行辅助



## MACcrash文件分析

mac程序崩溃时，会弹出一个窗口，给用户提示。用户可以直接在弹出的窗口中查看崩溃的信息。

这个.crash文件，同时会保存在 ~/Library/Logs/DiagnosticReports/

如下图所示

![奔溃](https://demo-1252736716.cos.ap-shanghai.myqcloud.com/%E5%B4%A9%E6%BA%83%E6%96%87%E4%BB%B6%E5%A4%B9.png)

.crash文件是一个文本文件。可以用文本编辑器直接打开。

要分析.crash文件。需要注意一些信息：


1、崩溃的线程

2、崩溃线程的堆栈

![堆栈](https://demo-1252736716.cos.ap-shanghai.myqcloud.com/%E5%A0%86%E6%A0%88.png)

3、崩溃的线程中，

第一列是从下到上的调用顺序

第二列是调用所在的动态库或程序

第三列是 当时程序虚拟内存中调用代码段的地址。 如果第四列未符号化，就是第四列两个数值的相加

第四列是 调用位置的具体符号+偏移。如果找不到符号，则是虚拟内存中代码段起始地址 + 偏移地址



## 如何进行符号转化

在mac程序编译时，为了减少程序的体积大小。就会把很多的符号信息，生成一个同名的.app.dSYM文件。如果没有.app.dSYM文件。是无法进行符号化的。

这一点和windows的pdb文件一样。（如果没有.app.dSYM文件，可以直接反编译源文件。找到对应的崩溃地址。但是太复杂）


现在我们就需要，如何把崩溃的地址，转换为符号。

一、xcode中有一个symbolicatecrash的脚本程序，配合dSYM文件，可以把为符号化的.crash文件转换为符号化的.crash文件

但是，这个脚本对于mac的程序不友好，使用时基本都会出错。所以放弃

二、使用atos来进行符号化。atos是一个类似于address2line的程序。

我们可以下面的命令，来进行符号化
```
atos --arch x86_64 -o ***.dSYM/Contents/Resources/DWARF/**-l  addr1  addr2
```

其中，–arch指定平台 -o指定dSYM文件， -l 指定崩溃地址   addr1 是崩溃程序虚拟内存代码段的起始地址，addr2是崩溃具体位置的虚拟内存代码段地址。

直接这么说可能比较难理解。对应到上图的最后一个位置，对应的命令为

```
atos --arch x86_64 -o MAXHUB.app.dSYM/Contents/Resources/DWARF/MAXHUB -l  0x10c21e000  0x000000010c256369
```

使用atos，就可以逐行把崩溃的信息翻译出来。


## APP程序和dSYM文件的标识

如何对崩溃的crash文件和一个dSYM文件做对应呢。在crash文件的Binary Images中：

![image](https://demo-1252736716.cos.ap-shanghai.myqcloud.com/image-uuid.png)

找到我们程序的那一行找到<>中的那个UUID，就是crash的UUID。

在包含.dSYM文件的目录中，执行：

```
mdfind -onlyin . UUID
```

就可以找到对应的.dSYM文件


