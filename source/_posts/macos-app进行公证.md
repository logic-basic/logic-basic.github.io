---
title: macos app进行公证
date: 2020-09-09 19:08:29
tags: [macos]
---

## 前言

自从macos的系统升级到10.14之后，mac开始对系统上的应用程序进行了大规模的限制。

* 应用必须拥有正规签名，才可以进行运行。
* 应用如果不进行公证，那么在第一次打开的时候，则会提示不信任的开发者。不能直接打开。需要在设置里点击仍然打开字样。

这样的话，会对用户的使用，造成很大的不便。


## 如何公证

### 使用Xcode

在工具栏选择archives, 弹出窗口后，选择自己要公证的程序。
选择distribute app, 然后进行公证。

但是使用xcode来公证，需要使用xcode编译才可以，Qt的程序就不能使用这种方法


### 使用命令进行公证

这里我们主要介绍这种方法

#### 上传

公证的命令是altool，需要xcrun altool来运行

```
xcrun altool --notarize-app --file **.zip  -u "appid" -p "passwd" --primary-bundle-id "bundle-id"
```

参数中，–file指定需要公证的app，需要先用zip进行打包，上传的时候，不支持直接指定app文件
-u 是指定apple id， -p 是指定密码  --primary-bundle-id 是指定app的 bundle id


zip 需要把app进行打包，可以用如下命令：

```
ditto -c -k --keepParent *.app *.zip
```

如果不想使用明文密码。可以先把密码保持到mac本地的钥匙串中，然后使用 -p "@keychain:AC_PASSWORD" 来进行
其中，AC_PASSWORD为iterm name。
添加密码进钥匙串，可以使用以下命令：

```
security add-generic-password -a "AC_USERNAME" -w "AC_PASSWORD" -s "ITEM_NAME"
```

其中，AC_USERNAME是用户名，对应altool的-u参数，AC_PASSWORD是真正的密码，ITEM_NAME是名称，填在altool的keychain:后面

添加成功后如图：

![](https://demo-1252736716.cos.ap-shanghai.myqcloud.com/macos%E8%AE%A4%E8%AF%811.png)

成功话，会显示如下信息：

No errors uploading '**.zip'.
RequestUUID = 64d2e096-ae50-4fea-97a0-4fe4c1c5fb85


#### 查询

上传应用后，需要等一段时间，来等待苹果来认证。不会超过一个小时，大概率在5分钟内完成。

在这期间，可以使用如下命令来查询状态

```
xcrun altool --notarization-info UUID -u "***" -p "@keychain:AT_PASSWD"
```

UUID是上传之后，返回的UUID，同时再附上用户密码即可。

正在处理中，返回如下：

```
No errors getting notarization info.
 
       Date: 2020-08-04 08:50:45 +0000
RequestUUID: 6c1e724f-491a-4273-b066-55c1b42c8e79
     Status: in progress
```

如果失败，返回如下：

```
No errors getting notarization info.
 
Date: 2020-08-04 08:50:45 +0000
Hash: a2c83b32c1214cc07558c32c71efdbf09bb19191e33c59fb65d7d74b7123104d
LogFileURL: https://osxapps-ssl.itunes.apple.com/itunes-assets/Enigma114/v4/b0/02/0a/b0020ada-f552-fadb-52bc-cea612a5a43e/developer_log.json?accessKey=1596725559_4297314406524191325_8NZckopz%2BOMwbzl%2BzTJubTgcQREE05cRlEU9iSG33MjRJHpebCD36Ve53DMtftKnJ70kDznYhES%2Bcek%2B5Fm07GsYDGbDowAYhHJL8Iez1I7q6kYOfGvIFPXJDdiZX923C4csk33brcaOD8kep8QtmXAsLl44xNeEODSc32r3qzo%3D
RequestUUID: 6c1e724f-491a-4273-b066-55c1b42c8e79
Status: invalid
Status Code: 2
Status Message: Package Invalid
```

可以通过LogFileURL的链接，来查看具体失败内容。


成功的话，会返回如下：

```
No errors getting notarization info.
 
Date: 2020-08-04 08:58:36 +0000
Hash: c3306b0409ca39b8b882a41789282270850a09f0f6442c73703596c47309e16b
LogFileURL: https://osxapps-ssl.itunes.apple.com/itunes-assets/Enigma114/v4/cc/df/3d/ccdf3d51-62c1-e5bd-25d6-2b03e34011d4/developer_log.json?accessKey=1596726032_3967876306215616611_GdX8pAnRtS7nHuM1g%2BbAKGp9SZ2StHfo7ALENG0dIm06WbJ01QLv20INy%2Fvs55v48bxRIy6h3d4nP0feDlVMGhb08D0r57qI7cmoFb55TQ%2B7lKuCOcIFqy18iv2Okn22pxaim2zhBAosAbf4xVYWe127mi7aahrrrZqHHcdRzNk%3D
RequestUUID: 06a30247-1490-459d-a9fd-1f42918a78f8
Status: success
Status Code: 0
Status Message: Package Approved
```

无论是否成功还是失败，apple都会发送一封邮件来告知你结果。


#### 查看app是否被公证

成功之后，需要稍等一段时间，公证会自动生效

可以通过`spctl -a -v  **.app`  来查询状态

如果出现：

```
**.app: accepted
source=Developer ID
```

说明还未公证成功，如果出现如下则证明公证成功

```
**.app: accepted
source=Notarized Developer ID
```


官方文档参考地址：

https://developer.apple.com/documentation/xcode/notarizing_macos_software_before_distribution/customizing_the_notarization_workflow#3087720




