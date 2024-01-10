---
title: 使用Django版本4及5建置圖片上傳與展示的平台
categories: develop
tags:
  - django
  - python
  - web
  - platform
abbrlink: '76205532'
date: 2024-01-09 00:00:00
---

![django-logo](https://i.imgur.com/X6Azqqr.png)

## 前言
----------

最近突然想到以前有寫一篇關於用Django上傳圖片的文章
但[那篇文章](https://natlee.github.io/Blog/posts/2800319203/)用的版本很舊了，所以決定以新版的測試並再寫一篇

<!--more-->

## 準備材料

我使用的Django版本是4.2.9，資料庫則是使用預設的sqlite
可以在命令列中輸入以下指令安裝需要的套件

```bash
pip install Django==4.2.9
```

避免過程中找不到資料夾位置
我把完成整個流程後的資料結構先放在這邊

```
.
├── db.sqlite3
├── img_test
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── migrations
│   │   ├── 0001_initial.py
│   │   ├── __init__.py
│   ├── models.py
│   ├── templates
│   │   ├── img_upload.html
│   │   └── show_img.html
│   ├── tests.py
│   ├── urls.py
│   └── views.py
├── manage.py
├── media
│   └── test-img
│       └── <YOUR_UPLOADING_IMAGES>
└── test_site
    ├── __init__.py
    ├── asgi.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

## 使用Django建立專案


1. 首先，先在命令列中創建一個Django的專案，名為`test_site`。
```bash
django-admin startproject test_site
```

2. 再移動到專案資料夾內，建立一個app。
```bash
cd test_site
python manage.py startapp img_test
```

3. 進到`test_site/test_site`內，找到`settings.py`

    1. 在install app的列表中最後一項新增剛建立的app。

        ```py
        INSTALLED_APPS = [
            'django.contrib.admin',
            'django.contrib.auth',
            'django.contrib.contenttypes',
            'django.contrib.sessions',
            'django.contrib.messages',
            'django.contrib.staticfiles',
            'img_test' # <- 寫在這邊
        ]
        ```

    2. 在`TEMPLATES`中的`context_processors`中新增一行，讓Django能夠處理media
        ```py
            TEMPLATES = [
            {
                'BACKEND': 'django.template.backends.django.DjangoTemplates',
                'DIRS': [],
                'APP_DIRS': True,
                'OPTIONS': {
                    'context_processors': [
                        'django.template.context_processors.debug',
                        'django.template.context_processors.request',
                        'django.contrib.auth.context_processors.auth',
                        'django.contrib.messages.context_processors.messages',
                        "django.template.context_processors.media", # <- 加入這行
                    ],
                },
            },
        ]
        ```

    3. 在`settings.py`的最後面新增以下兩行，讓Django能夠把圖片存到media資料夾中。
       這邊可以注意的是，新版Django的`BASE_DIR`本身的type是`pathlib.PosixPath`，所以在組合時可以用`/`。

        ```py
        MEDIA_URL = '/media/'
        MEDIA_ROOT = BASE_DIR / MEDIA_URL # <- MEDIA_ROOT是用BASE_DIR去組合的
        ```


        關於`MEDIA_URL`及`MEDIA_ROOT`的[官方說明](https://docs.djangoproject.com/en/5.0/ref/settings/#std-setting-MEDIA_ROOT)如下：

            ```txt
            MEDIA_ROOT¶
            Default: '' (Empty string)
            Absolute filesystem path to the directory that will hold user-uploaded files.
            Example: "/var/www/example.com/media/"

            MEDIA_URL¶
            Default: '' (Empty string)
            URL that handles the media served from MEDIA_ROOT, used for managing stored files. It must end in a slash if set to a non-empty value. You will need to configure these files to be served in both development and production environments.
            If you want to use {{ MEDIA_URL }} in your templates, add 'django.template.context_processors.media' in the 'context_processors' option of TEMPLATES.
            Example: "http://media.example.com/"
            ```



4. 進到剛建立的app資料夾`test_site/img_test`中，並修改`models.py`檔，新增以下class。
```py
class Image(models.Model):
    # upload_to指定的 `test-img` 為圖片上傳的路徑，不存在就創建一個新的。
    # 這邊要注意到時候會產生的路徑是 media/test-img
    img_url = models.ImageField(upload_to='test-img')
```

5. 在命令列中，執行下列指令以連結類別與資料庫的關係。
```bash
python manage.py makemigrations && python manage.py migrate
```

6. 進到app資料夾`test_site/img_test`中，並於`views.py`新增上傳圖片及顯示圖片的方法。
```py
from django.shortcuts import render
from img_test.models import Image

def upload_image(request):
    # 上傳圖片用的route
    # 使用POST方法去上傳圖片
    if request.method == 'POST':
        img = Image(img_url=request.FILES.get('img')) # <- 從使用者上傳的檔案中取得key叫做img的內容
        img.save() # <- 儲存圖片
    return render(request, 'img_upload.html')

def show_image(request):
    # 顯示圖片用的route
    imgs = Image.objects.all() # <- 取得所有圖片
    context = {
        'imgs' : imgs # <- 把圖片放到context中，並給予一個名稱叫做imgs
    }
    return render(request, 'show_img.html', context)

```

7. 再進到`test_site/img_test`中，新增一個`urls.py`檔案並新增html template及載入圖片的對應路徑。

```py
from django.urls import path
from img_test import views
urlpatterns = [
    path('upload', views.upload_image), # <- 上傳圖片的route
    path('show', views.show_image), # <- 顯示圖片的route
]
```

8. 進到專案資料夾`test_site/test_site`中，並於`urls.py`新增上傳圖片及顯示圖片的路徑。
```py
from django.contrib import admin
from django.urls import path
from django.urls import include
from django.conf import settings # <- 新增的
from django.conf.urls.static import static # <- 新增的

urlpatterns = [
    path('admin/', admin.site.urls), # <- 這行是預設的，可以不用刪掉
    path("", include("img_test.urls"), name="img_test"), # <- 新增的路徑
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT) # <- 新增的media route
```

其中，爲了讓Django能夠處理media，需要在`urlpatterns`的最後面新增一行，並且引入`settings`及`static`。


9. 建立一個`templates`資料夾在`test_site/img_test`

> 注意這邊一定要叫做`templates`

10. 在`test_site/img_test/templates`裡面新增上傳(`img_upload.html`)及展示(`show_img.html`)網頁的模板。

> 注意：這邊是用Server Side Render的方式去處理圖片

* img_upload.html
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

* show_img.html
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

11. 在命令列中，於專案資料夾底下輸入以下指令便可執行Django的伺服器。

```bash
python manage.py runserver
```

此時出現下列文字時，便可以進到網頁中進行圖片上傳與觀看了。
```bash
Django version 4.2.9, using settings 'test_site.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

可訪問的位置：
* 上傳圖片
> http://localhost:8000/upload

* 顯示圖片
> http://localhost:8000/show


## 實際展示
----------

* 上傳圖片
> http://localhost:8000/upload

先到上傳圖片的頁面，按下選擇檔案挑好圖片並按送出。
![upload-page](https://i.imgur.com/cVPGz03.png)

因為是簡易頁面，所以在上傳成功時，畫面中並不會有任何提示。

這時，你會看到命令列出現下列資訊。
```bash
[10/Jan/2024 07:18:25] "POST /upload HTTP/1.1" 200 397
```
上傳返回200，代表圖片已經上傳成功

* 顯示圖片
> http://localhost:8000/show

再到展示的頁面就可以看到剛剛上傳的照片了。
![show-page](https://i.imgur.com/9vumG0w.png)

```bash
[10/Jan/2024 07:42:36] "GET /show HTTP/1.1" 200 177
[10/Jan/2024 07:42:36] "GET /media/test-img/egypt_cat.jpg HTTP/1.1" 200 36413
```
在命令列可以看到返回200代表載入頁面並下載圖片成功。


## 結語
----------

這次實作是為了更新之前的文章，所以這次實作沒有太多的擴充

之前用的Django是2版，現在最新的已經是5版了

新版的Django在處理上傳圖片大致上還是差不多的

不過設定裏面，新版的路徑存取方式已經都改成`pathlib`的方式了

誠摯的建議不管是不是Django，在Python中使用路徑存取時，都可以使用`pathlib`的方式去處理

這樣除了可以避免在不同作業系統中，路徑的斜線不同而造成的錯誤之外，也可以讓程式碼更加的簡潔

另外，這次跟上次的實作是用Server Side Render（SSR）的方式去處理圖片

但現在大部分網站都是前後端分離的，所以圖片的處理方式也會不太一樣

有些會轉成base64存入database，有些還是會直接存取url

這邊就不多做介紹了，有興趣的話，可以去看看Google相片是怎麼做處理的（[System Design — Backend for Google Photos](https://mecha-mind.medium.com/system-design-backend-for-google-photos-e0abcd74dd36)）


## 參考資料
----------

- [使用Django版本2去建置圖片上傳與展示的平台](https://natlee.github.io/Blog/posts/2800319203/)
- [django 实现图片上传和显示操作](https://blog.csdn.net/c_beautiful/article/details/79755368)
- [Django上傳並顯示圖片](https://hk.saowen.com/a/025706bf034bd83359bffe190ab6cf5fc62398a67bc0589b4552b6e33b165c34)


---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E5%89%8D%E8%A8%80-de423396bb7f)

