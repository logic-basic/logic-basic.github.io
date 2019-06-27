---
title: Linux程序兼容.二
date: 2018-08-29 14:26:08
tags: Linux
---

在上一节说了如何对不同版本gcc进行适配。
这一节主要是外部静态库编译的配置记录。

以下的库都是编译32位的版本，如果需要64位只需要把-m32去掉即可

#### udev

仓库地址：https://git.kernel.org/pub/scm/linux/hotplug/udev.git

```
wget https://git.kernel.org/pub/scm/linux/hotplug/udev.git/snapshot/udev-173.tar.gz

tar -zxvf udev-173.tar.gz
cd udev-173

mkdir build
cd build
export CFLAGS=-m32

../configure --prefix=`pwd` --disable-udev_acl --disable-gudev --disable-keymap --enable-static
make
make install
```

#### x264

官网：https://www.videolan.org/developers/x264.html

```
wget http://download.videolan.org/pub/videolan/x264/snapshots/last_stable_x264.tar.bz2
tar -jxvf last_stable_x264.tar.bz2

../configure --prefix=`pwd` --enable-static --enable-shared 
make
make install
```

值得注意的是，x264在config里并没有提供编译32位的选项。如果本机是64，但是要编译32位的x264。需要手动改写configure脚本
在 706 行，进行位数判断时，加入`host_cpu=i386`强行使用32位编译

#### SDL2

```
cd 

mkdir build
cd build
../configure --prefix=`pwd` --disable-libudev --disable-dbus --disable-ime \
             --disable-ibus --disable-fcitx --disable-x11-shared  --disable-input-tslib \
             --disable-jack-shared --disable-pulseaudio-shared --disable-alsa-shared \
             --disable-video-wayland --disable-alsa --disable-audio --disable-video-x11-xcursor \
             --disable-video-x11-scrnsaver --disable-video-x11-vm --disable-video-x11-xdbe \
             --disable-video-x11-xinerama --disable-directfb-shared --disable-arts-shared \
             --disable-pulseaudio --disable-3dnow --disable-arts --disable-cpuinfo \
             --disable-dummyaudio --disable-ssemath --disable-power  --without-gnu-ld \ 
             --with-pic --enable-static --enable-shared CFLAGS=-m32
make 
make install
```

#### SDL_Image2 

```
wget https://www.libsdl.org/projects/SDL_image/release/SDL2_image-2.0.3.zip
unzip SDL2_image-2.0.3.zip

cd SDL2_image-2.0.3
mkdir build
cd build

../configure --prefix=`pwd` --disable-png-shared --disable-jpg-shared --disable-tif-shared --disable-webp --with-sdl-prefix=/home/dangjiahe/project/SDL2-2.0.8/build LDFLAGS=-L/home/dangjiahe/project/libpng-1.6.35/build/install/lib --enable-static CFLAGS=-m32
make
make install
```

#### libpng

```
wget 
```

#### libxcb

```
wget https://xcb.freedesktop.org/dist/libxcb-1.13.tar.gz
tar -zxvf libxcb-1.13.tar.gz 

cd libxcb-1.13
mkdir build
cd build

../configure --prefix=`pwd` --enable-static --enable-shared CFLAGS=-m32
make
make install
```

#### libXdmcp

X11相关库的地址： https://www.x.org/releases/individual/lib/

```
wget https://www.x.org/releases/individual/lib/libXdmcp-1.1.1.tar.gz
tar -zxvf libXdmcp-1.1.1.tar.gz

cd libXdmcp-1.1.1
mkdir build
cd build

../configure --prefix=`pwd` --enable-static --enable-shared CFLAGS=-m32
make
make install
```

#### libXau

```
wget https://www.x.org/releases/individual/lib/libXau-1.0.7.tar.gz
tar -zxvf libXau-1.0.7.tar.gz

cd libXau-1.0.7
mkdir build
cd build

../configure --prefix=`pwd` --enable-static --enable-shared CFLAGS=-m32
make
make install
```

#### libX11

```
wget https://www.x.org/releases/individual/lib/libX11-1.6.5.tar.gz
tar -zxvf libX11-1.6.5.tar.gz

cd libX11-1.6.5
mkdir build
cd build

../configure --prefix=`pwd` --enable-static --enable-shared CFLAGS=-m32
make
make install
```

#### libXi

```
wget https://www.x.org/releases/individual/lib/libXi-1.7.8.tar.gz
tar -zxvf libXi-1.7.8.tar.gz

cd libXi-1.7
mkdir build
cd build

../configure --prefix=`pwd` --enable-static --enable-shared CFLAGS=-m32
make
make install
```

#### libXest

```
wget https://www.x.org/releases/individual/lib/libXext-1.3.3.tar.gz
tar -zxvf libXext-1.3.3.tar.gz

cd libXext-1.3.3
mkdir build
cd build

../configure --prefix=`pwd` --enable-static --enable-shared CFLAGS=-m32
make
make install
```

#### libXtst

```
wget https://www.x.org/releases/individual/lib/libXtst-1.2.3.tar.gz
tar -zxvf libXtst-1.2.3.tar.gz

cd libXtst-1.2.3 
mkdir build
cd build

../configure --prefix=`pwd` --enable-static --enable-shared CFLAGS=-m32
make
make install
```

#### libz

```
wget https://zlib.net/zlib-1.2.11.tar.gz
tar -zxvf zlib-1.2.11.tar.gz

cd zlib-1.2.11
mkdir build
cd build

export CFLAGS=-m32
../configure --prefix=`pwd` --static
make
make install
```

