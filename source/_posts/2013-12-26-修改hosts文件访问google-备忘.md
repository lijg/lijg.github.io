---
title: 2013-12-26-修改hosts文件访问google(备忘)
date: 2013-12-26 17:09
tags:
categories: 杂
---

下载Android代码经常连不到google，将以下代码加入/etc/hosts中，解决网址不能访问问题
```
#code.google.com 
173.194.72.102 google.com

# Download
173.194.72.102 dl.google.com
173.194.72.136 dl-ssl.google.com

#Groups
74.125.31.101  groups.google.com

#Google URL Shortener
74.125.31.113  goo.gl

#Google App Engine
74.125.235.206 appengine.google.com

#Android Developer
74.125.235.200 developer.android.com

#for download android source
74.125.31.82 www.googlesource.com
74.125.31.82 android.googlesource.com
203.208.46.172 cache.pack.google.com
59.24.3.173 cache.pack.google.com
```
