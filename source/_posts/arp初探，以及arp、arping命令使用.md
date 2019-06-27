---
title: arp初探，以及arp、arping命令使用
date: 2019-03-05 17:20:33
tags: [网络,arp]
---

#### arp 协议简介

> arp (Address Resolution Protocol) 地址解析协议, 是通过解析网路层地址来找寻数据链路层地址的一个在网络协议包中极其重要的网络传输协议。

arp，可以通过ip地址，来获取对应IP的MAC地址.
不过arp也可以探测当前局域网是否有这个IP。


#### arp工具使用

arp会发一个广播，说，谁拥有某个IP，谁有的话，告诉我一下。

我们可以通过
```
arp -a
```
来查看当期arp的缓存表。但是，这个表并不是实时的。如果有一个设备拿到ip之后，并没有通信。在arp表中是找不到这个设备的。
并且，在执行这条指令时，本机也不会发起arp广播。只是查询本地的缓存。



如果, 我想要发起arp广播，用什么方法呢。可以使用：
```
arping x.x.x.x
```
arping会发起arp广播，在局域网内查找是否有该ip。如果有，则会有返回。如果没有，就会timeout。


#### arping 和 ping 的区别

1. 使用的协议不同, ping使用ICMP, arping使用arp
2. 有些windows配置会禁用ICMP，所以使用ping是没有办法检测设备存在的。
3. 如果本地的ip地址和你想检测远端的IP地址相同，ping会ping到自己。arping可以检测到远端是否有和你相同IP的设备。

