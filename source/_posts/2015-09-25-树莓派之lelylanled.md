---
title: 树莓派之lelylanled
date: 2015-09-25 17:34:08
tags: 
- 树莓派
categories: 
- 硬件
---

之前的网络呼吸灯里，树莓派相当于一个提供LED控制的服务器，客户端用的是curl命令行工具。客户端其用命令行、web、手机APP等都可以访问。

[lelylan](http://www.lelylan.com/)提供了一套开发环境和一套标准的接口，可以实现各种形式的客户端，比如`Device Web Component`、`Angular JS Client`、`Ruby Client`、`Node.js Client`、`Python Client`、`iOS Client`、`Android Client`以及`Lelylan Dashboard`等。

本文准备使用`Lelylan Dashboard`，在网络呼吸灯的基础上，加上真正的使用网页控制LED。


打开[Lelylan Dashboard](http://lelylan.github.io/devices-dashboard-ng/)，新建一个名为webled的Light设备。步骤如下：

1. Step 1
![Step 1](/images/2015-09-25/sadgjaw39u8ik1.png)

2. Step 2
![Step 2](/images/2015-09-25/sadgjaw39u8ik2.png)

3. Step 3
![Step 3](/images/2015-09-25/sadgjaw39u8ik3.png)

4. Step 4
![Step 4](/images/2015-09-25/sadgjaw39u8ik4.png)

5. Step 5
![Step 5](/images/2015-09-25/sadgjaw39u8ik5.png)


`lelylan`的包每次会以`PUT`方式发送到指定的URL，包的内从可以从`request.json`中获取`dict`格式的信息，各种信息的格式如下(其中的id是lelylan自动分配的，可能有所不同)：

**打开LED**
```
{
    u'properties': [
        {
            u'id': u'518be88300045e0610000008',
            u'value': u'off'
        }
    ]
}
```

**关闭LED**
```
{
    u'properties': [
        {
            u'id': u'518be88300045e0610000008',
            u'value': u'off'
        }
    ]
}
```

**设置亮度为31(可选范围0~100)**
```
{
    u'properties': [
        {
            u'id': u'518be8b100045e99cc000004', 
            u'value': u'31'
        }
    ]
}
```


**设置颜色**
```
{
    u'properties': [
        {
            u'id': u'518be90500045e1521000008',
            u'value': u'#cc4c92'
        }
    ]
}
```

**设置闪烁频率(0.0~20)**
```
{
    u'properties': [
        {
            u'id': u'518be95500045e1521000009', 
            u'value': 1
        },
        {
            u'id': u'518be9aa00045e0610000009',
            u'value': u'10'
        }
    ]
}
```

**设置呼吸频率(ms)**
```
{
    u'properties': [
        {
            u'id': u'5497e32513437b4a6b000002', 
            u'value': 10
        }
    ]
}
```


**设置呼吸灯颜色**
```
{
    u'properties': [
        {
            u'id': u'518be90500045e1521000008',
            u'value': u'#3d3d3d'
        }
    ]
}
```

了解了`lalylan`的包格式之后，下一步就是如何在`Flask`中解析这些命令，并用来控制LED。

比如，处理LED的打开和关闭事件
```
@app.route("/LED", methods=['GET', 'PUT', 'POST'])
def LED():
    if request.method == 'GET':
        return "Hello LelylanLED!"
    elif request.method == 'PUT':
        # 解析lelylan发过来的参数
        properties = request.json['properties']
        id0 = properties[0]['id']
        value0 = properties[0]['value']

        # LED on/off 的ID
        if id0 == ID_LED_ONOFF:
            # 根据value值判断LED动作
            if value0 == VAL_LED_ON:
                breath.set(100)
            elif value0 == VAL_LED_OFF:
                breath.set(0)
            else:
                abort(404)
        else:
            abort(404)
        return value0
    else:
        return abort(404)
```

[完整代码](https://github.com/ljgabc/raspberrypi/blob/master/lelylanled.py)