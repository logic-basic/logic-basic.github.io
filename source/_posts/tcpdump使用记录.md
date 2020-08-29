---
title: tcpdump使用记录
date: 2019-09-02 19:49:15
tags: 网络
---

##### 指定固定网卡

```
tcpdump -i xxx
```

##### 指定发往目标IP

```
tcpdump dst host xxx.xxx.xxx.xxx
```

##### 指定来源IP

```
tcpdump src host xxx.xxx.xxx.xxx
```

##### 指定两个主机之间的通信

```
tcpdump ip host xxx.xxx.xxx.xxx and xxx.xxx.xxx.xxx
```

##### 保存信息到文件

```
tcpdump -w xxx.cap
```

