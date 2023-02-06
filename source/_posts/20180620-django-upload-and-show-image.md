---
title: 使用Django框架建置圖片上傳與展示的平台
categories: develop
tags:
  - django
  - python
  - web
  - platform
abbrlink: 2800319203
date: 2018-06-20 00:00:00
---

![](https://i.imgur.com/PECvJrh.jpg)
<center>*ネトゲの嫁は女の子じゃないと思った？*</center>

## 前言
----------

今天一張UR都沒抽到
所以在這邊整理了一個如何製作Django建置圖片上傳平台教學

<!--more-->

## 準備材料
----------

我使用的Python版本是3.6.5，以及Django版本是2.0.6
處理圖片可能需要使用到pillow套件，資料庫則要用到mysqlclinent
可以在命令列中輸入以下指令安裝需要的套件

```bash
    pip install Django==2.0.6
    pip install pillow
    pip install mysqlclient
```

避免過程中找不到資料夾位置
我把完成整個流程後的資料結構先放在這邊

    testsite
    │   db.sqlite3
    │   manage.py
    ├─  imgTest
    │  │   admin.py
    │  │   apps.py
    │  │   models.py
    │  │   tests.py
    │  │   views.py
    │  │   __init__.py
    │  └─  migrations
    │       │   __init__.py
    │       └─  0001_initial.py
    ├─  media
    │  └─  img
    │       └─  XXXXX.jpg
    ├─  templates
    │  │   imgUpload.html
    │  └─  showImg.html
    └─  testsite
        │   settings.py
        │   urls.py
        │   wsgi.py
        └─  __init__.py


## 使用Django建立專案
----------

1.  首先，先在命令列中創建一個Django的專案，名為testsite。
```bash
    django-admin startproject testsite
```

2.  再移動到專案資料夾內，建立一個app。
```bash
    cd testsite
    python manage.py startapp imgTest
```

3.  (1)  進到testsite/testsite內，找到settings.py，並在install app的列表中最後一項新增剛建立的app。
```py
    INSTALLED_APPS = (
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'imgTest',
)
```
    (2)  然後修改settings.py檔，在上面新增mysql的資訊、圖片上傳的路徑資訊。
```py
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'imgs',
            'USER': 'root',
            'PASSWORD': '',
            'HOST': 'localhost',
            'PORT': 3306
        }
    }
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media').replace('\\', '/')
    MEDIA_URL = '/media/'
```
    (3) 在修改settings.py檔中，TEMPLATES的路徑資訊，其中的DIRS項增加一個templates。
```py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': ['templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

4.  進到剛建立的app資料夾testsite/imgTest中，並修改models.py檔，新增以下class。
```py
    class IMG(models.Model):
        # upload_to為圖片上傳的路徑，不存在就創建一個新的。
        img_url = models.ImageField(upload_to='img')
```

5.  在命令列中，執行下列指令以連結類別與資料庫的關係。
```bash
    python manage.py makemigrations
    python manage.py migrate
```

6.  進到app資料夾testsite/imgTest中，並於views.py檔新增上傳圖片及顯示圖片的方法。
```py
    # 上傳圖片
    def uploadImg(request):
        if request.method == 'POST':
            img = IMG(img_url=request.FILES.get('img'))
            img.save()
        return render(request, 'imgupload.html')

    # 顯示圖片
    def showImg(request):
        imgs = IMG.objects.all()
        context = {
            'imgs' : imgs
        }
        return render(request, 'showImg.html', context)

```

7.  再進到testsite/testsite中，修改urls.py檔案，並新增Html template及載入圖片的對應路徑，如下。
```py
    from django.contrib import admin
    from django.urls import path
    from django.conf import settings
    from django.conf.urls.static import static
    from imgTest.views import uploadImg, showImg
    urlpatterns = [
        path('admin/', admin.site.urls), # admin
        path('uploadImg/', uploadImg), # 上傳圖片的地方
        path('showImg/', showImg) # 顯示圖片的地方
    ] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

8.  直接在專案資料夾testsite/底下建立templates資料夾，並在裡面新增上傳及展示網頁的模板。

* imgUpload.html
```html
    <!DOCTYPE html>
    <head>
        <meta charset="UTF-8">
        <title>Upload</title>
    </head>
    <body>
        <form action="" method="post" enctype="multipart/form-data">
            {% csrf_token %}
            <input type="file" name="img">
            <input type="submit" value="Upload">
        </form>
    </body>
    </html>
```
* showImg.html
```html
    <!DOCTYPE html>
    <head>
        <meta charset="UTF-8">
        <title>展示圖片</title>
    </head>
    <body>
        {% for item in imgs %}
            <img src="{{ item.img_url.url }}"><br/>
        {% endfor %}
    </body>
    </html>
```

9.  在命令列中，於專案資料夾底下輸入以下指令便可執行Django的伺服器。
```bash
    python manage.py runserver
```
    此時出現下列文字時，便可以進到網頁中進行圖片上傳與觀看了。
```bash
    Django version 2.0.6, using settings 'testsite.settings'
    Starting development server at http://127.0.0.1:8000/
    Quit the server with CTRL-BREAK.
```


## 實際展示
----------

* 上傳圖片
> http://localhost:8000/uploadImg/

先到上傳圖片的頁面，按下選擇檔案挑好圖片並按送出。
![](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Django_Upload_and_Show_Image/01.png)

因為是簡易頁面，所以在上傳成功時，畫面中並不會有任何提示。

這時，你會看到命令列出現下列資訊。
```bash
    [20/Jun/2018 06:38:52] "GET /uploadImg/ HTTP/1.1" 200 422
    Not Found: /favicon.ico
    [20/Jun/2018 06:38:53] "GET /favicon.ico HTTP/1.1" 404 2319
    [20/Jun/2018 06:39:37] "POST /uploadImg/ HTTP/1.1" 200 422
```
意思是一開始進去網頁載入頁面，然後找不到favicon的icon圖片
最後一行則返回200，代表圖片已經POST成功。

* 顯示圖片
> http://localhost:8000/showImg/

再到展示的頁面就可以看到剛剛上傳的照片了。
![](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Django_Upload_and_Show_Image/02.png)

```bash
    [20/Jun/2018 06:40:04] "GET /showImg/ HTTP/1.1" 200 344
    [20/Jun/2018 06:40:04] "GET /media/img/riko.jpg HTTP/1.1" 200 65002
```
在命令列可以看到返回200代表載入頁面並下載圖片成功。

## 結語
----------

因為網路上Django跟Python的版本不一
除了Python版本造成不相容外
Django版本也有很大的影響
所以很多地方相關的範例都有些小錯誤，沒辦法執行
希望在這邊做成一篇完整且能執行的版本後能夠幫助到大家
順便記錄一下，以後自己忘記也能提醒自己如何操作

## 參考資料
----------

[[django 实现图片上传和显示操作](https://blog.csdn.net/c_beautiful/article/details/79755368)]
[[Django上傳並顯示圖片](https://hk.saowen.com/a/025706bf034bd83359bffe190ab6cf5fc62398a67bc0589b4552b6e33b165c34)]
