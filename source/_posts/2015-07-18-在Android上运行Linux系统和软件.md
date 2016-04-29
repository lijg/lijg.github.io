---
title: 在Android上运行Linux系统和软件
date: 2015-07-18 16:22:46
tags: 
categories: 
- Android
---

`chroot`，即`change root directory`(更改`root`目录)。在`linux`系统中，系统默认的目录结构都是以`/`，即是以根(root)开始的。而在使用`chroot`之后，系统的目录结构将以指定的位置作为`/`位置。

在`Android`系统中运行`Linux`系统的核心就是`chroot`命令。

**步骤**

### 1 准备文件系统

将文件系统拷贝到Android系统中，如`/sdcard/linuxfs`

```
> ls /sdcard/linuxfs
bin  dev  etc  linuxrc  lib  proc  root  sbin  sys  tmp  usr  var
```

### 2 准备工作

```
mount -o bind /dev/ /sdcard/linuxfs/dev/
mount -t proc proc /sdcard/linuxfs/proc/
mount -t sysfs sysfs /sdcard/linuxfs/sys/
```

### 3 chroot

```
busybox chroot /sdcard/linuxfs /linuxrc
或
busybox chroot /sdcard/linuxfs /bin/sh
```
完。