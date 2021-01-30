---
title: cmake 编译mac Big Sur universal库
date: 2020-11-11 19:56:12
tags: [cmake , macos]
---

## 前言

在11月11日, 苹果发布了新款Macbook 电脑，统一采用ARM架构的M1芯片。
具体的性能就不详细介绍了。最大的变化是从x86架构转移到了ARM架构。
那么之前的app和库，就不能完美的运行。


## Universal App

Universal App 是苹果推出的一个过渡方案，在使用xcode构建ap时，会自动编译两套二进制程序.
一套x86, 一套arm64。然后把两个程序打包在一起。这样无论在哪个架构的电脑上运行，都能顺利的运行程序.

这一过程，是xcode自动执行的，但是同时也需要app所依赖所有的库也必须是 Universal版本. 
如果库只有一个架构，那么在构建打包时，将会出错。

Universal App构建，需要在xcode12.2之后的版本才被支持


## 使用命令行构建

如果构建app，xcode会很方便，但是构建库时，使用xcode就比较复杂了。
我们可以使用如下makefile直接编译目标的架构程序

```
x86_app: main.c
    $(CC) main.c -o x86_app -target x86_64-apple-macos10.12
arm_app: main.c
    $(CC) main.c -o arm_app -target arm64-apple-macos11
universal_app: x86_app arm_app
    lipo -create -output universal_app x86_app arm_app
```

这也是从官方说明中，摘抄出来的，网址如下：
https://developer.apple.com/documentation/xcode/building_a_universal_macos_binary?language=objc



## 使用cmake构建通用库

使用裸命令来编译，非常复杂，需要使用lipo来进行合并。这对我们原有的库的编译方式影响很大。

cmake提供了一个宏，可以用来同时编译两个架构的代码。

这个宏就是CMAKE_OSX_ARCHITECTURES.

```
cmake .. "-DCMAKE_OSX_ARCHITECTURES=arm64;x86_64"
```

通过这个指令，cmake在编译的时候，会加入`-arch arm64 -arch x86_64`这个参数。
这样，就只需要一次编译，就可以编译出两个架构的通用库

可以使用file命令来对二进制文件进行检查，通货版本输出如下:

```
client: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64]
client (for architecture x86_64):	Mach-O 64-bit executable x86_64
client (for architecture arm64):	Mach-O 64-bit executable arm64
```

需要注意的是，该方法对cmake版本和xcode版本都有要求：
cmake -  3.11版本以上
xcode -  12.2版本以上


## xcode编译器版本切换

一台mac电脑中，是可以同时存在不同xcode版本的，这样也是利于开发人员进行开发和调试

比如clang编译器就可以通过 xcode-select来切换版本

```
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer

或者

sudo xcode-select -s /Applications/Xcode-Beta.app/Contents/Developer
```

注意，如果两个xcode放在同一个文件夹，需要给他们分配不同的名称。
如果存在两个版本的xcode，需要先用xcode-select切换到12.2之后的版本，才可以用cmake来编译通用版本库


