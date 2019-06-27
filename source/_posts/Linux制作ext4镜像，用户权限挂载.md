---
title: Linux制作ext4镜像，用户权限挂载
date: 2018-09-08 09:52:55
tags: [Linux , 镜像制作]
---

## Linux制作ext4格式的镜像文件

- 1 生成指定大小的存储文件

```
dd if=/dev/zero of=system_new.img bs=1024 count=10240
```

其中，if是输入文件，of是输出文件，bs是一次读写缓冲区的字节大小，count是总共多少次。
文件的总大小就是`bs * count`，在本例中，大小就是1024 * 10240。总大小为10M。

执行成功如下：

```
记录了10240+0 的读入
记录了10240+0 的写出
10485760 bytes (10 MB, 10 MiB) copied, 0.0231433 s, 453 MB/s
```

- 2 格式化文件为ext4 

这一步也是最重要的一步，我们使用mkfs.ext4来格式化，这个工具大部分linux系统都会有：

```
mkfs.ext4 -L my_ios -E root_owner=1000:1000 system_new.img
```

其中，-L 和 -E 的解释如下：

> -L new-volume-label
>      Set the volume label for the filesystem to new-volume-label.  The maximum length  of  the  volume  label  is  16
>      bytes.
>
> -E extended-options
>      root_owner[=uid:gid]
>          Specify  the  numeric  user and group ID of the root directory.  If no UID:GID is specified, use the
>          user and group ID of the user running mke2fs.  In mke2fs 1.42 and earlier the UID  and  GID  of  the
>          root  directory  were set by default to the UID and GID of the user running the mke2fs command.  The
>          root_owner= option allows explicitly specifying these values, and avoid side-effects for users  that
>          do not expect the contents of the filesystem to change based on the user running mke2fs.

-L 是给我们的镜像文件制定一个卷标名称。如果不指定的话，在挂载我们镜像的时候，显示的将是一个长串的uuid。

-E 是指定额外选项，这里我们设置root_owner。因为我们希望挂载的时候，挂载目录为我们的用户权限。所以要通过他来设置uid和gid。
   1000则是我们电脑默认的用户。

至此，格式化完成。

- 3 创建临时文件夹，并挂载。向镜像文件写数据。

到这一步，其实我们的镜像文件已经制作好了，现在需要向我们的镜像文件中写数据。

创建临时文件夹

```
mkdir temp_folder
```

挂载镜像到临时文件夹
 
```
sudo mount -o loop system_new.img ./temp_folder
```

拷贝我们的到文件到挂载的目录下

```
cp hello.sh ./temp_folder
```

删除lost+found文件夹

```
sudo rm -rf ./temp_folder/lost+found
```

- 4 弹出我们的挂载

```
sudo umount temp_folder
```

至此，我们的镜像就已经制作完成。


## Remarks 

使用mkfs.ext4格式化U盘也是同样的办法。如果格式化时不加参数，那么挂载的时候。U盘写文件将没有权限。

