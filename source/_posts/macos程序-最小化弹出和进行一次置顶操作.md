---
title: macos程序-最小化弹出和进行一次置顶操作
date: 2020-08-21 19:36:22
tags: [macos, tip]
---

在查找这两个方法的过程中走了很多弯路，特此记录

## 把已经最小化的程序弹出


```
for(NSWindow* win in [NSApp windows]) {
    if([win isMiniaturized]) {
        [win deminiaturize:self];
    }
}
```


## 把已经置于底部的macos程序置于顶层

```
[NSApp activateIgnoringOtherApps:YES];
[NSApp setActivationPolicy:NSApplicationActivationPolicyRegular];
```
