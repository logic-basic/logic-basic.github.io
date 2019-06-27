---
title: Mac设置环境变量
date: 2018-05-08 19:43:50
tags: mac 
---

在我们平时使用和开发软件时，通常需要配置各种参数，比如环境变量。
比如，我们设置PATH环境变量，来直接运行程序。

在windows中，我们可以通过电脑->属性来配置环境变量。这个环境变量是全局生效的。
并且重启电脑依然有效。

在MacOs系统中，我们有多重方式配置环境变量。

### 手动export变量

在终端中，我们可以使用export设置环境变量，设置后只对当前终端有效。如果开启新的终端，则会失效。

### 写入`.bash_profile`

当一个新的终端开启时，系统会自动加载`.bash_profile`中的内容。
所以我们可以在`.bash_profile`中export我们的变量。

### 使用launchctl指令

前面的两个方法，都只能在终端中生效，在MacOS系统里，我们有很多的GUI程序，为了让环境变量对GUI程序也生效。

e.g. `launchctl setenv DEBUG_MODE true` 

使用上面的命令，我们设置了一个名为`DEBUG_MODE`的变量，该变量值为true
如果我们要取消，则使用unsetenv

e.g. `launchctl unsetenv DEBUG_MODE`

值得注意的是，这种设置方法，在电脑重启后，变量会失效。
不过，MAC电脑一般很少重启就是了。

PS: 以上测试系统为: macOS High Sierra 版本: 10.13.4

