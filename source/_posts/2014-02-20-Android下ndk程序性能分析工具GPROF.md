---
title: Android下ndk程序性能分析工具GPROF
date: 2014-02-20 14:50:10
tags: gprof ndk
categories: Android
---

由于oprofile在imx6上bug多多，很不好用，因此又找到了新的性能测试工具gprof。

已经有人将其移至到android平台，可以用来测试ndk程序的性能，见[android-ndk-profiler](https://code.google.com/p/android-ndk-profiler/)

#使用步骤

1 下载程序，源码在jni目录，编译 

2 编译和链接自己的程序时时加上 -pg选项

3 在自己的程序中添加 
```
// start dump
monstartup("your_lib.so");

...

// stop dump
moncleanup();
```

4 运行程序，默认生成/sdcard/gmon.out，里面保存了dump的记录

5 将gmon.out下载到本地，使用arm-linux-androideabi-gprof工具分析

/path/to/arm-linux-androideabi-gprof your_lib.so gmon.out
