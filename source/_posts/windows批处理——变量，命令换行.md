---
title: windows批处理——变量，命令换行
date: 2018-05-09 17:29:51
tags: [windows , 批处理]
---

我们知道，在linux系统中，我们可以使用shell脚本来进行重复，批量任务的处理。
同样，在windows系统中，也有相应的工具，那就是批处理

批处理本质也是文本文件，后缀为`.bat`
可以直接双击bat文件运行，或在windows的cmd命令终端中执行

今天记录两个有关批处理的内容，变量和批处理的换行


### 变量

#### 如何设置变量

在批处理文件中，设置变量要使用set指令，如下：

```
set PRODUCT_NAME="hello world"

echo %PRODUCT_NAME%
```

我们设置了一个名为`PRODUCT_NAME`变量，它的内容为"hello world"。
所以，终端会输出`"hello world"`

#### 设置变量的本质

值得注意的是，set设置也是环境变量。比如，我们可以通过set PATH变量来设置程序运行的路径。
不过，通过set设置的变量和linux中export是一样的，只在当前终端生效，当终端退出时，设置的变量将失效。

#### 设置变量的生效范围

刚才，我们说，set本质和export一样。他设置了临时的环境变量。
但是，有时候，我们可能不需要他在整个终端中都生效，只想让他在局部的文本中使用。

现在，我们可以使用`setlocal`和`endlocal`来限定我们变量生效的范围。
e.g.

```
set G_NAME="Lisa"

setlocal
set NAME="Jack"
echo %NAME%
endlocal

echo %G_NAME%
echo %NAME%
```

输出如下：

```
"Jack"
"Lisa"
%NAME%
```
当出了endlocal之后，在里面设置的变量将无效。

### 命令的换行 

在写批处理脚本时，我们可能会遇到比较长的指令语句。比如说`cmake`后跟它的配置参数。
在linux系统，如果命令没写完，我们可以使用`\`来换行，但是，在windows里不一样。
在windows系统中，我们使用`^`来代替`\`

e.g.
```
cmake .. -DCMAKE_BUILD_TYPE=Debug ^
         -DCMAKE_TOOLCHAIN_FILE=${TOOLCHAIN_FILE} ^
         -DBUILD_SHARED_LIBS=ON 
```

