---
title: NSUserDefaults 文件储存位置
date: 2020-12-23 15:18:44
tags: macos
---

## NSUserDefaults

NSUserDefaults 是app来存储一些用户配置项的。方便程序下次启动来读取

NSUserDefaults 会在本地电脑生成一个plist文件, 该plist会以app的bundle id来命名

## NSUserDefaults文件位置

plist文件的位置在 `~/Library/Preferences`
