---
title: Shadowsocks加速github git clone的方法
date: 2018-03-26 12:54:48
tags: 小工具 
---

1. 首先配置好Shadowsocks代理，保证可以使用Shadowsockets。

2. 在命令行终端里，执行如下命令:

```
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

注意： 如果是MAC平台，而且安装的是NG版本的小飞机，把上面的1080端口改为1086即可。

3. 现在即可使用http进行加速了。

注意： clone 时，需要选择https协议。

4. 如果要取消加速，运行如下：

```
git config --global --unset http.proxy
git config --global --unset https.proxy
``` 
