---
title: windows切换分辨率接口使用
date: 2018-09-07 17:29:46
tags: windows
---

## 使用windows api来自动切换屏幕分辨率

- 1 引入使用的头文件

```C++
#include <windows.h>
#include <winuser.h>
```

- 2 获取显示设备基本信息

```C++
DISPLAY_DEVICE device;
DWORD device_num = 0;
memset(&device, 0, sizeof(DISPLAY_DEVICE));
device.cb = sizeof(DISPLAY_DEVICE);

bool founded = false;
EnumDisplayDevices(NULL, device_num, &device, EDD_GET_DEVICE_INTERFACE_NAME)); 
```

- 3 获取具体显示器的具体参数

```C++
DEVMODE dev_mode;
dev_mode.dmSize = sizeof(DEVMODE);

EnumDisplaySettings(device.DeviceName, ENUM_CURRENT_SETTINGS, &dev_mode);
```

- 4 设置要改变的值

```C++
dev_mode.dmPelsWidth = 1920;
dev_mode.dmPelsHeight = 1080;
dev_mode.dmPosition = {0, 0};
dev_mode.dmFields = (DM_BITSPERPEL | DM_PELSWIDTH | DM_PELSHEIGHT | DM_POSITION);
```

- 5 改变屏幕分辨率

```C++
ChangeDisplaySettingsEx(device.DeviceName, &dev_mode, NULL, 0, NULL);
```

