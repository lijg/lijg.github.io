---
title: 2014-01-21-修改Android设备中USB设备的默认权限
date: 2014-01-21 21:06:40
tags: 
- USB
categories: 
- Android
---

在Android设备中，有时想简单粗暴地将所有usb设备的权限设置成777（这样任何应用都不用权限就可以使用USB设备了），方法如下：

修改源码中``$(ANDROID_SRC)/system/core/rootdir/ueventd.rc``，将
```
/dev/bus/usb/*            0660   root       usb
```
改为
```
/dev/bus/usb/*            0777   root       usb
```

即可。
