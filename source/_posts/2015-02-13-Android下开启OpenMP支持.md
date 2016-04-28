---
title: Android下开启OpenMP支持
date: 2015-02-13 20:29:08
tags: OpenMP
categories: Android
---

Android NDK自R9开始支持OpenMP，不需要额外的库文件和头文件，只需要在编译时加上如下的编译选项即可：
```
CFLAGS += -fopenmp

CXXFLAGS += -fopenmp

LDFLAGS += -fopenmp
```