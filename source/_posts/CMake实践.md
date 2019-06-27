---
title: CMake实践
date: 2018-07-23 16:46:49
tags: cmake
---

## 什么是cmake

提到cmake，我们就不得提到编译，提到编译，就~~

无证驾驶:

```Shell
gcc -o main main.cpp
```

新司机:

```Shell
./configure
make
```

老司机:

```Shell
cmake ..
make
```

和其他编译工具一样，cmake帮我们生成和管理我们的软件。

cmake 是 "Cross platform Make" , 而不是 "C Language Make" 

cmake 是一个自动化构建系统。用来管理软件编译及构建。并且不依赖与特定编译器或系统。

那么，总结一下，cmake就是帮我们构建软件的工具。

## why cmake 

为什么要使用cmake呢

1. cmake是跨平台的工具，并且可以使用该平台最方便的工具。
2. cmake文件易于编辑和理解

第一，make 并不会直接构建出最中的软件，而是说明一个标准的构建流程。在不同的平台下，生成各个平台的构建档。 
比如，linux生成Makefile，Macos生成Makefile或Xcode工程，Windows下生成VS工程。再由这些构建档生成出最终的软件。
这个也是cmake最大的特色。

因为我们库需要全平台的使用，并且需要适配该平台最通用的运行库(RuntimeLib)。
比如，在linux下生成so。在windows下生成基于VCRuntime的lib。在macos下生成.a或.dylib
在这一点，cmake是十分具有优势，因为它可以生成各个平台自己的工程文件。

第二，cmake的语法十分简洁，易于记忆和理解。

主要是基于以上的两点原因，我们选择使用cmake

## 获取cmake

### windows

1. 登录https://cmake.org/download/  下载最新release版本
2. 安装cmake，并加入PATH环境变量

### MacOS

```Shell
brew install cmake
```

### Linux

#### ubuntu

```Shell
sudo apt install cmake
```

### arch

```Shell
sudo pacman -S cmake
```

PS: 较低版本的ubuntu可能会安装低于3.0版本的cmake。目前我们的项目需要cmake版本最低为3.0。

## cmake使用

### 通用构建

cmake 的描述文件是`CMakeLists.txt`。
在该文件中，我们告诉cmake如何构建我们的项目。

1. 我们打开一个由cmake构建的项目, 其目录结构如下:

```Shell
CMakeLists.txt out            src            test           toolchain
```

2. 创建一个文件夹，用来存放构建产生的文件

```Shell
mkdir build
cd build
```

3. 使用cmake生成构建

```Shell
cmake ..
```

4. 使用cmake编译

```Shell
cmake --build .
```

这是一种在各个平台都可以使用的方法, `cmake ..` 会生成各个平台默认的构建。
linux和Macos默认为 Makefile，windows默认为VS工程。
无论使用在哪个平台，都可以使用`cmake --build .` 来生成最终的软件。

note: 以上的例子，都是没有指定debug和release的，在实际的使用中，windows需要注意区分

### 指定生成目标工程文件--Xcode工程

上面讲了，macos系统默认生成的makefile，如果我们需要在mac上进行更好的调试，可以使用cmake生成Xcode工程。

第一步和第二步和之前相同。

3. 指定生成Xcode工程

```Shell
cmake .. -GXcode
```

4. 打开Xcode工程进行调试

## 开始一个自己的cmake项目

在学习如何使用cmake生成软件后，我们现在开始自己的cmake项目。

首先，我们要先写好我们需要构建的源文件，这里展示一个demo。

```
.
└── src
    ├── execute
    │   └── main.cpp
    └── lib_file
        ├── lib.cpp
        └── lib.h
```

其中，main.cpp内容如下:

```c++
  
   #include <iostream>
  
   int main(void) {
       std::cout << "Hello world" << std::endl;
       return 0;
   }
```

lib.cpp:

```c++
  
   #include <iostream>
  
   void LibPrint() {
       std::cout << "Lib: Hello world" << std::endl;
   }
  
```

lib.h:

```c++
  
   #pragma once
  
   void LibPrint();
```

在有了源文件之后，我们开始编写CMakeLists.txt，在src的同级目录下创建该文件。

CMakeLists.txt:

```CMake
 
  # 要求cmake的版本不低于3.0
  cmake_minimum_required(VERSION 3.0)
 
  # 将src/execute目录下的源文件加入到 MAIN_SRC 这个变量中
  aux_source_directory(./src/execute MAIN_SRC)
  # 将src/lib_file目录下的源文件加入到 LIB_SRC 这个变量中
  aux_source_directory(./src/lib_file LIB_SRC)
 
  # 构建demo为可执行程序，文件为 MAIN_SRC 中所包含的文件
  add_executable(demo ${MAIN_SRC})
  # 构建demo_static_lib为静态库，文件为 LIB_SRC 中所包含的文件
  add_library(demo_static_lib STATIC ${LIB_SRC})
  # 构建demo_shared_lib为动态库，文件为 LIB_SRC 中所包含的文件
  add_library(demo_shared_lib SHARED ${LIB_SRC})
 
```

通过这几行简单的代码，我们就可以获取到一个可执行文件和两个库文件。
详细的内容已经写到注释上，这里就不进行过多的解释。
完成后，按照上面的步骤，就可以生成相应的程序。

## 变量

和其他的众多脚本或构建器一样，cmake同样拥有变量。
例如上一节中的，MAIN_SRC和LIB_SRC。就是把源文件名称放入变量里。

此外，我们如何设置一个新的变量，和Windows有些类似。我们使用set:

```CMake
set (Hello 1)
```

我们设置了一个名为 Hello 的变量，它的值为1。

在cmake系统中，为我们设置了很多的默认变量，就是说，这些名称的变量默认有值, 比如：

```CMake
${CMAKE_SOURCE_DIR} #表示源文件的顶层目录
```

还有一些变量，用来配置编译器或它的参数，比如：

```CMake
${CMAKE_C_FLAGS}     # C的编译选项
${CMAKE_CXX_FLAGS}   # C++编译选项
```

有一些用来判读编译时的系统：

```CMake
${APPLE}
${UNIX}
${WIN32}
${ANDROID}
```

如果当前环境符合上述变量，变量有值非空，否则，变量值为空

## 打印信息

有时候，我们需要查看变量的值。这时候，就需要打印一下变量看看。
变量打印使用message，比如：

```CMake
# 打印项目根路径
message(${CMAKE_SOURCE_DIR})

# 新建变量Hello值为1
set (Hello 1)
# 打印出1
message(${Hello})
```

在执行`cmake .. `之后，路径信息会被打印出来

### 条件判断

在cmake中，条件判断的格式如下:

```CMake
if(xxx)
    # 执行内容
endif()
```

if和endif相对应。并且后面都要跟括号。if的括号里为条件的内容

## 多平台的处理

我们使用CMake是为了编译在各个平台上的程序，有时候，我们使用了一些和平台相关的内容，就需要就行条件判断来处理。

判断windows系统:

```CMake
if (WIN32)
    # 处理
endif()
```

判断MacOS系统:

```CMake
if (APPLE)
    # 处理
endif()
```

判断编译android程序:

```CMake
if (ANDROID)
    # 处理
endif()
```

判断编译Linux程序:

```CMake
if (UNIX AND NOT ANDROID AND NOT APPLE)
    # 处理
endif()
```

第一，在这里，我们看到，Linux的判断比较特殊，首先判断UNIX系统，之后排除掉ANDROID和APPLE的情况。
之后剩下的就是Linux系统了。因为，我们大部分使用Linux系统编译android程序，同时APPLE也是基于UNIX的系统。
所以，我们这样判断，cmake是不能直接判断Linux系统的。


第二，我们发现，我们没有使用C语言中`== && ||`等操作符，这些在cmake中是不能使用的，代替的是字母单词`AND, NOT , match`等等。


## CMakefile.txt -- install

在生成我们可执行文件，或者库之后。还剩最后一步 —— install 。
通常，我们生成的文件会在我们创建的build文件夹中，我们生成库之后，需要把响应的文件整理起来，保存到一个地方。

在uinx系统中，就是make install.
在cmake中，就有这样的功能，并且，cmake在windows上会生成一个INSTALL的项目，来完成install的功能。

我们在之前的CMakeLists.txt中加入install描述:

```CMake
  # 设置安装路径为根文件夹下的out文件夹
  set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/out)

  # install demo 到out/bin下
  INSTALL(TARGETS demo DESTINATION bin)
  # install demo_static_lib 到out/lib下
  INSTALL(TARGETS demo_static_lib DESTINATION lib)
  # install demo_shared_lib 到out/bin下
  INSTALL(TARGETS demo_shared_lib DESTINATION bin)
 
  # install lib.h头文件 到out/include下
  INSTALL(FILES ./src/lib_file/lib.h DESTINATION include)
```

这样，在编译完之后，可以执行make install。或者 INSTALL项目来完成项目的安装。

## CMakeLists 多目录结构

在cmake中，可以进行多目录构建。及可以把CMakeLists.txt放到各个子目录中，最后用一个顶层的CMakeLists.txt去包括他们。

```
.
├── CMakeLists.txt
├── README.md
└── src
    ├── execute
    │   ├── CMakeLists.txt
    │   └── main.cpp
    └── lib_file
        ├── CMakeLists.txt
        ├── lib.cpp
        └── lib.h
```

类似于这样。我们只需要在顶层的CMakeLists.txt(和src目录同级)中加入两句话：

```CMake
add_subdirectory(./src/execute)
add_subdirectory(./src/lib_file)
```

就可以引入底层的两个CMake文件

## CMake链接库

在开发中，不可避免。我们要使用到别人或者第三方库。这就需要我们cmake对库进行链接。

这里，我们用上面的例子进行试验，用demo去链接demo_static_lib静态库。
在`add_executable`后面加入一句话：

```CMake
target_link_libraries(demo demo_static_lib)
```

只要要这一句话，就可以让demo程序链接demo_static_lib这个静态库

PS： 值得注意的是，在Linux下，如果链接了多个静态库。如果库之间有依赖，那么对库的链接顺序是有要求的。
比如，我们使用了A库，A库又使用了pthread。那么我们必须先链接A库，再链接pthread。否则会报错。

```CMake
target_link_libraries(demo A phtread)  # 正确，可以链接成功
target_link_libraries(demo phtread A)  # 错误，链接会失败
```


更多详细内容可以google cmake官网进行内容查找。

