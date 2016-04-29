---
title: 移植OProfile到Android
date: 2014-01-18 21:44:40
tags: 
- Oprofile 
- 性能
categories:
- Android
---

1 第一次编译popt

*下载popt-1.14，编译安装*
```
ac_cv_va_copy=yes ./configure --with-kernel-support --target=arm-linux-gnueabi --host=arm-linux-gnueabi --enable-install-libfd --prefix=/home/lijg/oprofile/popt
make
make install
```

*下载binutils-2.20，编译安装*
```
./configure --with-kernel-support --target=arm-linux-gnueabi --host=arm-linux-gnueabi --enable-install-libbfd --prefix=/home/lijg/oprofile/binutils --disable-nls --enable-install-libiberty --enable-shared
make
make install
```

*下载最新的oprofile,编译安装*
```
./configure --with-kernel-support --target=arm-linux-gnueabi --host=arm-linux-gnueabi --with-binutils=/home/lijg/oprofile/binutils --prefix=/home/lijg/oprofile
make
make install
```

此时，通过file查看生成的opreport等文件，发现还是通过动态链接库生成的，不是静态的，在Android下不好使用，需要将其生成静态二进制程序

2 重新编译oprofile

以ophelp为例，通过file查看ophelp，看到是*dynamically linked (uses shared libs)*
```
cd ~/oprofile-0.9.9/utils/
file ophelp
    <font color="gray">ophelp: ELF 32-bit LSB executable, ARM, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.31, BuildID[sha1]=0x0d1e63861273943a2c92bd61a3d6ce92163f9088, not stripped</font>
```

将ophelp删除，在utils目录执行make，最后一行link，可以看到生成ophelp的命令
```
rm ophelp
make
    <font color="gray">/bin/sh ../libtool --tag=CC   --mode=link arm-linux-gnueabi-gcc -W -Wall -fno-common -Wdeclaration-after-statement -g -O2 -L/data/oprofile/lib -Xlinker -R -Xlinker /data/oprofile/lib  -o ophelp ophelp.o ../libop/libop.a ../libutil/libutil.a -lpopt -liberty -ldl</font>
    <font color="gray">libtool: link: arm-linux-gnueabi-gcc -W -Wall -fno-common -Wdeclaration-after-statement -g -O2 -Wl,-R -Wl,/data/oprofile/lib -o ophelp ophelp.o  -L/data/oprofile/lib ../libop/libop.a ../libutil/libutil.a /data/oprofile/lib/libpopt.so -liberty -ldl -Wl,-rpath -Wl,/data/oprofile/lib -Wl,-rpath -Wl,/data/oprofile/lib</font>
```

将最后一样手动改为静态编译（去掉所有的链接到libpopt.so libiberty.so libbfd.so的语句，改为使用libpopt.a libiberty.a libbfd.a），最后再加上--static
```
arm-linux-gnueabi-gcc -W -Wall -fno-common -Wdeclaration-after-statement -g -O2 -Wl,-R -Wl,/data/oprofile/lib -o ophelp ophelp.o  ../libop/libop.a ../libutil/libutil.a /data/oprofile/lib/libpopt.a /data/oprofile/lib/libiberty.a -ldl -Wl,-rpath -Wl,/data/oprofile/lib -Wl,-rpath -Wl,/data/oprofile/lib --static
```

查看生成的文件，看到是statically linked, 在Android可以直接使用
```
file ophelp
    <font color="gray">ophelp: ELF 32-bit LSB executable, ARM, version 1 (SYSV), statically linked, for GNU/Linux 2.6.31, BuildID[sha1]=0xe78959fba10954b8bc68de2e32bb09ba3678a19a, not stripped</font>
```

按照同样的方法，重新编译op-check-perfevents, oprofiled，opannotate, oparchive, opgprof, opreport, operf, opimport, opjitconv，代码如下

*op-check-perfevents*
```
cd ~/oprofile-0.9.9/utils/
rm op-check-perfevents
arm-linux-gnueabi-gcc -W -Wall -fno-common -Wdeclaration-after-statement -g -O2 -Wl,-R -Wl,/data/oprofile/lib -o op-check-perfevents op_check_perfevents-op_perf_events_checker.o  /data/oprofile/lib/libpopt.a /data/oprofile/lib/libiberty.a -ldl -Wl,-rpath -Wl,/data/oprofile/lib -Wl,-rpath -Wl,/data/oprofile/lib --static
```

*oprofiled*
```
cd ~/oprofile-0.9.9/daemon/
rm oprofiled
arm-linux-gnueabi-gcc -W -Wall -fno-common -Wdeclaration-after-statement -fno-omit-frame-pointer -g -O2 -o oprofiled init.o oprofiled.o opd_stats.o opd_pipe.o opd_sfile.o opd_kernel.o opd_trans.o opd_cookie.o opd_events.o opd_mangling.o opd_perfmon.o opd_anon.o opd_spu.o opd_extended.o opd_ibs.o opd_ibs_trans.o ../libabi/libabi.a ../libdb/libodb.a ../libop/libop.a ../libutil/libutil.a /data/oprofile/lib/libpopt.a /data/oprofile/lib/libiberty.a -ldl --static
```

*opannotate*
```
cd ~/oprofile-0.9.9/pp/
rm opannotate
arm-linux-gnueabi-g++ -W -Wall -fno-common -ftemplate-depth-50 -g -O2 -Wl,-R -Wl,/data/oprofile/lib -o opannotate opannotate.o opannotate_options.o common_option.o  -L/data/oprofile/lib ../libpp/libpp.a ../libopt++/libopt++.a ../libregex/libop_regex.a ../libutil++/libutil++.a ../libop/libop.a ../libutil/libutil.a ../libdb/libodb.a /data/oprofile/lib/libpopt.a /data/oprofile/lib/libbfd.a /data/oprofile/lib/libiberty.a -ldl --static
```

*oparchive*
```
cd ~/oprofile-0.9.9/pp/
rm oparchive
arm-linux-gnueabi-g++ -W -Wall -fno-common -ftemplate-depth-50 -g -O2 -Wl,-R -Wl,/data/oprofile/lib -o oparchive oparchive.o oparchive_options.o common_option.o  -L/data/oprofile/lib ../libpp/libpp.a ../libopt++/libopt++.a ../libregex/libop_regex.a ../libutil++/libutil++.a ../libop/libop.a ../libutil/libutil.a ../libdb/libodb.a /data/oprofile/lib/libpopt.a /data/oprofile/lib/libbfd.a /data/oprofile/lib/libiberty.a -ldl --static
```

*opgprof*
```
cd ~/oprofile-0.9.9/pp/
rm opgprof
arm-linux-gnueabi-g++ -W -Wall -fno-common -ftemplate-depth-50 -g -O2 -Wl,-R -Wl,/data/oprofile/lib -o opgprof opgprof.o opgprof_options.o common_option.o  -L/data/oprofile/lib ../libpp/libpp.a ../libopt++/libopt++.a ../libregex/libop_regex.a ../libutil++/libutil++.a ../libop/libop.a ../libutil/libutil.a ../libdb/libodb.a /data/oprofile/lib/libpopt.a /data/oprofile/lib/libbfd.a /data/oprofile/lib/libiberty.a -ldl --static
```

*opreport*
```
cd ~/oprofile-0.9.9/pp/
rm opreport
arm-linux-gnueabi-g++ -W -Wall -fno-common -ftemplate-depth-50 -g -O2 -Wl,-R -Wl,/data/oprofile/lib -o opreport opreport.o opreport_options.o common_option.o  -L/data/oprofile/lib ../libpp/libpp.a ../libopt++/libopt++.a ../libregex/libop_regex.a ../libutil++/libutil++.a ../libop/libop.a ../libutil/libutil.a ../libdb/libodb.a /data/oprofile/lib/libpopt.a /data/oprofile/lib/libbfd.a /data/oprofile/lib/libiberty.a -ldl --static
```

*operf*
```
cd ~/oprofile-0.9.9/pe_profiling/
rm operf
arm-linux-gnueabi-g++ -W -Wall -fno-common -ftemplate-depth-50 -g -O2 -Wl,-R -Wl,/data/oprofile/lib -o operf operf.o  -L/data/oprofile/lib ../libperf_events/libperf_events.a ../libutil++/libutil++.a ../libdb/libodb.a ../libop/libop.a ../libutil/libutil.a ../libabi/libabi.a /data/oprofile/lib/libiberty.a -ldl --static
```

*opimport*
```
cd ~/oprofile-0.9.9/libabi/
rm opimport
arm-linux-gnueabi-g++ -W -Wall -fno-common -ftemplate-depth-50 -g -O2 -Wl,-R -Wl,/data/oprofile/lib -o opimport opimport.o  -L/data/oprofile/lib libabi.a ../libdb/libodb.a ../libopt++/libopt++.a ../libutil++/libutil++.a ../libutil/libutil.a /data/oprofile/lib/libpopt.a /data/oprofile/lib/libiberty.a -ldl --static
```


*opjitconv*
```
cd ~/oprofile-0.9.9/opjitconv/
rm opjitconv
arm-linux-gnueabi-gcc -W -Wall -fno-common -Wdeclaration-after-statement -g -O2 -Wl,-R -Wl,/data/oprofile/lib -o opjitconv opjitconv.o conversion.o parse_dump.o jitsymbol.o create_bfd.o debug_line.o  ../libutil/libutil.a /data/oprofile/lib/libbfd.a /data/oprofile/lib/libiberty.a -ldl -Wl,-rpath -Wl,/data/oprofile/lib -Wl,-rpath -Wl,/data/oprofile/lib --static
```


3 最后附上生成的静态可执行文件
[oprofile-static-0.9.9](/_attachment/oprofilestatic.tar/bz2)