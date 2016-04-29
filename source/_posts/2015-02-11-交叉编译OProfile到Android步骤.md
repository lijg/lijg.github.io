---
title: 交叉编译OProfile到Android步骤
date: 2015-02-11 20:15:09
tags: 
- Oprofile
categories: 
- Android
---

# 1 第一次编译profile
## 1.1 编译popt-1.16
### 1.1.1 补丁

```
--- popt-1.14/popthelp.c    2008-03-27 13:33:08.000000000 -0400
+++ popt-1.14/popthelp.c    2011-02-01 21:33:21.000000000 -0500
@@ -15,6 +15,11 @@
 #include <sys/ioctl.h>
 #endif
 
+/* needed to find the struct winsize */
+#ifdef __ANDROID__
+#include <asm/termios.h>
+#endif
+
 #define    POPT_WCHAR_HACK
 #ifdef     POPT_WCHAR_HACK
 #include <wchar.h>         /* for mbsrtowcs */
```

```
--- popt-1.14/configure.ac  2008-04-02 13:17:39.000000000 -0400
+++ popt-1.14/configure.ac  2011-02-01 21:33:54.000000000 -0500
@@ -43,7 +43,7 @@
 
 AC_ISC_POSIX
 AM_C_PROTOTYPES
-AC_CHECK_VA_COPY
+
 
 AC_CHECK_HEADERS(float.h glob.h langinfo.h libintl.h mcheck.h unistd.h)
```

### 1.1.2编译
```
./configure --with-kernel-support --target=arm-linux-androideabi --host=arm-linux-androideabi --enable-install-libfd --prefix=$PWD/_install
make && make install
```
由于需要静态编译，删除_install/lib目录下的所有so,so.*等文件。

## 1.2编译binutils-2.20
### 1.2.1 补丁
```
diff -Nur binutils-2.23.2-old/libiberty/getpagesize.c binutils-2.23.2/libiberty/getpagesize.c
--- binutils-2.23.2-old/libiberty/getpagesize.c 2005-03-28 04:09:01.000000000 +0200
+++ binutils-2.23.2/libiberty/getpagesize.c 2013-10-24 22:45:24.000000000 +0200
@@ -20,6 +20,7 @@
 
 */
 
+#ifndef ANDROID
 #ifndef VMS
 
 #include "config.h"
@@ -88,3 +89,4 @@
 }
 
 #endif /* VMS */
+#endif /* ANDROID */
diff -Nur binutils-2.23.2-old/bfd/archive.c binutils-2.23.2/bfd/archive.c
--- binutils-2.23.2-old/bfd/archive.c   2013-03-25 09:06:19.000000000 +0100
+++ binutils-2.23.2/bfd/archive.c   2013-10-24 21:48:30.000000000 +0200
@@ -1863,7 +1863,9 @@
     {
       /* Assume we just "made" the member, and fake it.  */
       struct bfd_in_memory *bim = (struct bfd_in_memory *) member->iostream;
-      time (&status.st_mtime);
+      time_t t;
+      time (&t);
+      status.st_mtime = t;
       status.st_uid = getuid ();
       status.st_gid = getgid ();
       status.st_mode = 0644;
```
### 1.2.2 编译
```
./configure --with-kernel-support --target=arm-linux-androideabi --host=arm-linux-androideabi --enable-install-libbfd --prefix=$PWD/_install --disable-nls --enable-install-libiberty --enable-shared
make && make install
```
由于需要静态编译，删除_install/lib目录下的所有so,so.*等文件。

## 1.3编译oprofile

```
CFLAGS="-I/home/lijg/popt-1.16/_install/include"  CXXFLAGS="-I/home/lijg/popt-1.16/_install/include" LDFLAGS="-L/home/lijg/popt-1.16/_install/lib -static" ./configure --with-kernel-support --target=arm-linux-androideabi --host=arm-linux-androideabi --with-binutils=/home/lijg/binutils-2.22/_install --prefix=$PWD/_instal
make && make install
```