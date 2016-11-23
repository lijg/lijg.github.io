---
title: 使用Django和Blueimp Gallery制作照片墙
date: 2016-11-23 21:19:14
tags:
- Python
- Django
categories:
- Linux
---

本文记述了怎么使用`Django`和`Blueimp Gallery`一步一步搭建一个照片墙。

使用到的资源：
* 前端: Blueimp Gallery
* 后端: Django
* 缩略图: easy_thumbnails
* 部署: nginx + gunicorn + supervisor 

开发环境是Centos 7.2 + Python 2.7.5

运行效果：
![1](/images/2016-11-23/1.png)
![2](/images/2016-11-23/2.png)

## 1 创建项目
假设当前用户是`ljgabc`，当前目录是`/home/ljgabc`。
```
virtualenv django
source django/bin/activate
pip install django pillow easy_thumbnails gunicorn
django-admin startproject websites
cd websites
python manage.py startapp gallery
```

## 2 修改配置

首先修改`websites/settings.py`中的`INSTALLED_APPS`，添加应用，
```
INSTALLED_APPS = [
    ...,
    'gallery.apps.GalleryConfig',
    'easy_thumbnails',
]
```

其它配置如下：
* 图片文件上传位置: `gallery/uploads`
* 图片访问地址: `<siteurl>/uploads/xxxx.jpg`
* 缩略图大小为`75x75`
* 最终静态文件存放地址为`static/`
因此修改`websites/settings.py`，添加以下内容，
```
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

MEDIA_ROOT = os.path.join(BASE_DIR, 'gallery')
MEDIA_URL = '/'
IMAGE_PREFIX = 'uploads'

THUMBNAIL_ALIASES = {
  '': {
    '75x75' : {'size': (75,75), 'crop':True},
  },
}
```

## 3 配置URL
由于我们的网站只有一页，所以需要配置的URL只有`/`、`/admin`和`/uploads`，修改`websites/urls.py`，配置如下：
```
import os
from django.conf.urls import url, include
from django.contrib import admin
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^$', include('gallery.urls')),
]

urlpatterns += static(settings.IMAGE_PREFIX, document_root=os.path.join(settings.MEDIA_ROOT, settings.IMAGE_PREFIX))
```

添加`gallery/urls.py`,
```
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^$', views.gallery, name='gallery'),
]
```

## 4 添加模型
编辑`gallery/models.py`，添加`Image`模型，
```
from django.db import models
from django.conf import settings
from django.utils.encoding import python_2_unicode_compatible

# Create your models here.

@python_2_unicode_compatible
class Image(models.Model):
    '''
    模型包含Title、文件、创建时间等
    '''
    title = models.CharField(max_length=250, blank=True)
    original = models.ImageField(upload_to=settings.IMAGE_PREFIX, default='/tmp/none.jpg')
    created = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

## 5 添加视图
编辑`gallery/views.py`，配置视图，
```
from django.shortcuts import render
from .models import Image
# Create your views here.

def gallery(request):
    image_list = Image.objects.all()
    return render(request, 'gallery/index.html', {
        'image_list': image_list
        })
```

## 6 添加静态文件
从网上下载`Blueimp Gallery`文件，将其中的`css`、`img`和`js`文件夹放入`gallery/static/gallery/`文件夹下。

## 7 添加首页模板
创建`gallery/templates/gallery/index.html`，内容如下：
{% raw %}
```
{% load static %}
{% load thumbnail %}
<!DOCTYPE html>
<html lang="zh-cn">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Photo Gallery</title>
    <link rel="stylesheet" href="{% static '/gallery/css/blueimp-gallery.min.css' %}">
    <link rel="stylesheet" href="{% static '/gallery/style.css' %}">
  </head>
  <body>
    <!-- The Gallery as lightbox dialog, should be a child element of the document body -->
    <div id="blueimp-gallery" class="blueimp-gallery blueimp-gallery-controls">
        <div class="slides"></div>
        <h3 class="title"></h3>
        <a class="prev">‹</a>
        <a class="next">›</a>
        <a class="close">×</a>
        <a class="play-pause"></a>
        <ol class="indicator"></ol>
    </div>
    <div id="links" class="image-gallery">
      {% if image_list %}
      {% for image in image_list %}
        <a href="{{ image.original.url }}" title="{{ image.title }}">
            <img src="{{ image.original | thumbnail_url:'200x200' }}" alt="{{ image.title }}">
        </a>
      {% endfor %}
      {% endif %}
    </div>
    <script src="{% static '/gallery/js/blueimp-gallery.min.js' %}"></script>
    <script>
      document.getElementById('links').onclick = function (event) {
          event = event || window.event;
          var target = event.target || event.srcElement,
              link = target.src ? target.parentNode : target,
              options = {index: link, event: event},
              links = this.getElementsByTagName('a');
          blueimp.Gallery(links, options);
      };
    </script>
  </body>
</html>
```
{% endraw %}
## 8 初始化
```
python manage.py makemigrations gallery
python manage.py migrate
python manage.py createsuperuser
```

## 9 预览
```
python manage.py runserver 0.0.0.0:8000
```

## 10 配置`supervisor`
```
yum install supervisor
```

创建`/etc/supervisor.d/gallery.ini`, 添加以下内容：
```
[program:gallery]
command=/home/ljgabc/django/bin/gunicorn websites.wsgi:application
directory=/home/ljgabc/websites
user=ljgabc
autostart=true
autorestart=true
```

启动supervisor，
```
systemctl start supervisor
# supervisorctl start gallery
```

此时访问`127.0.0.1:8000`应该可以看到应用已经启动。

## 11 配置`nginx`

首先将所有用到的静态文件收集到`STATIC_ROOT`目录下，
```
python manage.py collectstatic
```

创建`/etc/nginx/conf.d/gallery.conf`,
```
server {
    listen 80;
    server_name www.your-domain-name.com;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:8000;
    }

    location /static/ {
        root /home/ljgabc/websites;
    }
}
```

启动`nginx`
```
systemctl start nginx
```
全文丸。