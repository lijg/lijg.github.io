---
title: Python序列化模块Pickle
date: 2015-06-04 20:50:52
tags: Python
categories: Linux
---

最近在写一个基于文件计数器，脚本每执行一次记一次数，计数结果在文件中存储，用来统计开发板开关机次数。

> `pickle`模块实现了一个简单的的Python对象序列化和反序列化的高效算法。

可以方便的存储和读取Python对象到文件中。

计数器代码如下：

```
#!/usr/bin/python
# -*- coding: utf-8 -*-

import pickle

FILE = r'counter.pickle'

def counter(step):
    '''基于文件的计数器，每调用一次，记step次数'''

    # 如果文件存在，则从文件读出，否则counter置为0
    try:
        with open(FILE, 'rb') as f:
            counter = pickle.load(f)
    except IOError:
        counter = 0

    # 计数
    counter += step

    # 将counter写入文件，不存在则创建
    try:
        with open(FILE, 'wb+') as f:
            pickle.dump(counter, f)
    except IOError:
        print 'Error Open ' + FILE + " for write"
        return -1

    return 0

```