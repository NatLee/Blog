---
title: 以Django建立Docker GUI控制界面
categories: dev
tags:
  - docker
  - django
  - python
abbrlink: 41a1fec1
date: 2021-01-02 00:00:00
---

![operation](https://github.com/NatLee/django-docker-gui/blob/main/doc/operation.gif?raw=true)

## 前言

在上一篇文章，[使用Docker快速建立GUI圖形化的深度學習開發環境](https://natlee.github.io/Blog/posts/c7016718/)中

最後結尾提到，我要來搞一個『以Django爲後端開發集成式管理的Docker開發環境』，於是就有了這篇文章

---

<!--more-->

## 內容

---

> 注意！這篇文章只是一個專案的介紹！

你可能以爲我們要搞個Django的管理後台，然後把Docker的指令包裝起來，讓使用者可以透過網頁來管理開發環境

![並沒有](https://i.imgur.com/R8kzXNx.png)

但是並沒有，因爲這個專案得分成兩個階段

第一個階段，我們得先搞個Django的管理後台，然後把Docker的指令包裝起來

讓使用者可以透過網頁來控制Docker

第二個階段才是把建立開發環境的Docker的指令包裝起來，讓使用者可以透過網頁來管理開發環境

### Stage I: Django Docker GUI

> 本專案可直接查看 [Django Docker GUI](https://github.com/NatLee/django-docker-gui)

大致整理一下要達成這個階段的目標，我們需要以下功能

![](https://i.imgur.com/OhQeeA9.png)

除了得透過Django定義API去控制裝置的Docker外

我們還要設計登入方式以及簡單的前端頁面來呈現

#### Authentication

登入的部分，我們採用JWT

這邊我們使用 [Simple JWT](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/) 來實作

基本上，我之前有寫過一個 Django 的 template

上面有很多標配功能，我們可以直接套用

> [Django Project Template](https://github.com/NatLee/django-project-template)

在前端的部分，我們需要一個簡易的登入頁面

![login-page](https://i.imgur.com/dB0yO62.png)

主要的程式碼可以參考[login.html](https://github.com/NatLee/django-docker-gui/blob/main/src/login/templates/login.html)

輸入帳號密碼登入之後，就可以取得 JWT token

#### Docker API

關於Docker的控制，我們使用已經包裝好的 [Docker SDK for Python](https://docker-py.readthedocs.io/en/stable/) 來實作

它的使用非常簡單，僅僅兩行就可以建立一個 Docker Client 的 instance

```python
import docker
client = docker.from_env()
```

#### Task Queue

> [Django RQ Task](https://github.com/NatLee/django-docker-gui/blob/main/src/xterm/task.py)

執行 Docker 指令的時候，我們會需要一些時間來等待 Docker 完成指令

這時候我們就需要一個 Task Queue 來處理這些指令

Survey一些選項，有些人用 celery 或是 Django Q 去控制任務

但我們其實只要一個簡易的 Task Queue 就可以了

所以後來我採用了 [Django RQ](https://github.com/rq/django-rq) 來實作

這邊從程式碼內拿一個例子來看，以下是一個建立 Docker Container 的任務

```python
@job
def run_image_task(image_id):
    # 先拿到 Docker Client 的 instance
    client = docker.from_env()
    # 直接使用這個instance去建立 Container
    container = client.containers.run(
        image_id, # 輸入的image id
        stdin_open=True, # 開啓stdin
        detach=True, # detach container
        tty=True, # 開啓tty
    )
    image_name = "none" # 預設image name，避免沒有image name的時候出錯
    if container.image.tags:
        # 如果有image name，就把image name拿出來
        image_name = container.image.tags[0]

    # 拿到container name
    container_name = container.attrs['Name'][1:]

    msg = f"Container [{container_name}] ({image_name}) has been created"

    message = {
        "action": "CREATED",
        "details": msg
    }
    # 透過WebSocket發送訊息
    send_notification_to_group(message=message)
    return msg
```

所以有了 Docker SDK ，我們控制 Docker 的方式就變得很容易

但這邊要注意的是`send_notification_to_group`

這個函數是用來發送 WebSocket 訊息的

在前端的部分，我們有很多地方需要透過 WebSocket 來接收訊息

#### WebSocket

![container-list](https://i.imgur.com/HuAZV0F.png)

登入之後的頁面，我們可以看到上面有一個 Container List 的表格

這個表格上的每一個 Container 都有一個操作欄位，裡面有一些按鈕

觸發這些按鈕之後，我們會需要一些時間來等待 Docker 完成指令（使用 Task Queue）

每個連線到這個頁面的瀏覽器就需要透過 WebSocket 來接收訊息，並且更新容器的狀態

如果點進去 Console ，會進到一個網頁版 CMD 的頁面

![web-console](https://i.imgur.com/8emo6Cf.png)

這個頁面也是透過 WebSocket 來接收訊息，並且更新 CMD 的輸出

![pull-image](https://i.imgur.com/R90BqTb.png)

在拉取 Image 的時候，我們也需要一個 WebSocket 來接收訊息

所以，websocket 在即時更新狀態上扮演了很重要的角色

- 後端實作

那麼，講了那麼多應用

在後端 Websocket 的部分，我採用 Django Channels 來實作

原本有考慮 Socket.IO ，但是它太特規

除了前後端套件版本都要符合才能用之外，還只能吃同一個path

有夠不友善的，所以就放棄了

相較之下，Django Channels 除了能夠支援前端的原生 WebSocket 之外

還可以以不同用途來使用不同的 path 來做 WebSocket

以這個專案來說，下面是我們的 WebSocket path 例子：

```python
from django.urls import re_path
from xterm import consumers

websocket_urlpatterns = [
    re_path(r"ws/console/$", consumers.ConsoleConsumer.as_asgi()), # <- 這個是 console 用的 consumer
    re_path(r"ws/pull-image/$", consumers.PullConsumer.as_asgi()), # <- 這個是 pull image 用的 consumer
    re_path(r'ws/notifications/$', consumers.NotificationConsumer.as_asgi()), # <- 這個是通知用的 consumer
]
```

可以看到，我們建立三個不同的 Consumer ，並且使用不同的 path 去接收 WebSocket 訊息

另外，在 [Task Queue](#task-queue) 的部分，我們使用了`send_notification_to_group`這個函數

它可以從 server 端去對已經建立連線的 websocket 廣播訊息

以下是這個函數的程式碼：

```python
from asgiref.sync import async_to_sync # 非同步函數轉同步函數
from channels.layers import get_channel_layer

def send_notification_to_group(message):
    channel_layer = get_channel_layer() # 取得channel layer

    # 因爲 `channel_layer.group_send` 是非同步函數
    # 所以我們需要把它轉成同步函數去執行
    async_to_sync(channel_layer.group_send)(
        # 在 websocket 中，每個頻道都有專屬的group name
        # 這邊我們傳送的訊息要給使用 `group_notifications` 作爲group name 的頻道
        'group_notifications',
        {
            'type': 'send_notification', # 這個是我們在 consumer 中定義的函數
            'message': message
        }
    )
```

這個函數會把訊息傳送給`group_notifications`這個頻道

並且它會透過在 consumer 中定義的函數`send_notification`把訊息傳送給所有連線的 websocket client

以下是 consumer 中的程式碼：

```python
class NotificationConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = 'notifications'  # Can be dynamic based on path
        self.room_group_name = f'group_{self.room_name}' # 這邊就是 group name

        # 把連線到這個 consumer 的 client 加入 group
        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )

        await self.accept()

    async def disconnect(self, close_code):
        # 結束連線時，把 client 從 group 中移除
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )

    # 這邊定義收到訊息時的處理函數
    async def send_notification(self, event):
        message = event['message'] # 從 event 中取得訊息

        # 將訊息用json格式dump後，傳送給 client
        await self.send(text_data=json.dumps({
            'message': message
        }))
```

這邊我們可以看到，當有 client 連線到這個 consumer 時

我們會把 client 加入 group，並且在 client 斷線時把 client 從 group 中移除

當 client 還在連線中的狀態下，我們就可以透過`send_notification`這個函數把訊息傳送給 client

在後端我們就算是完成了 WebSocket Server 的部分

- 前端實作

對於前端來說，我們要怎麼接收這些訊息呢？

> [Container Javascript](https://github.com/NatLee/django-docker-gui/blob/main/src/static/js/containers.js)

我們在前端的程式碼，可以使用原生的 WebSocket 來建立連線

並且在收到訊息時，透過`onmessage`這個函數來處理訊息

```js
function notificationWebsocket() {

    // 判斷 protocl 是 http 或是 https
    var ws_scheme = window.location.protocol == "https:" ? "wss" : "ws";
    // 組合 websocket path
    var ws_path = ws_scheme + '://' + window.location.host + "/ws/notifications/";
    // 使用原生的 WebSocket 來建立連線
    var socket = new WebSocket(ws_path);

    // ...

    socket.onmessage = function (event) {
        // 把收到的訊息轉成 JSON 格式
        const data = JSON.parse(event.data);
        console.log('Notification message:', data.message);
        let action = data.message.action;
        if (action === "WAITING") {
            // 如果是 WAITING 的訊息，就顯示訊息
            // createToastAlert 是彈跳視窗的通知函數
            createToastAlert(data.message.details, false);
            let containerID = data.message.data.container_id;
            // 拿到 record 的 DOM 元素
            let record = document.getElementById("actions-" + containerID);
            // 收到訊息後，把按鈕改成 Waiting（避免重複點擊）
            record.innerHTML = `
                <span class="btn btn-warning btn-sm me-3">Waiting</span>
            `;
        }
        if (action === "CREATED" || action === "STARTED" || action === "STOPPED" || action === "REMOVED") {
            // 如果是 CREATED, STARTED, STOPPED, REMOVED 的訊息，就顯示訊息
            createToastAlert(data.message.details, false);
            // 並且重新取得 container 的資訊
            fetchAndDisplayContainers();
        }

    };

    // ...
}
```

其中，`createToastAlert`這個函數可以參考下面的程式碼：

```js
function createToastAlert(msg, isFailure) {
    const toastId = Math.random().toString(36).substr(2, 16); // 對彈跳的視窗元素產生一個唯一的ID
    const toastColorClass = isFailure ? 'bg-danger' : 'bg-primary'; // 判斷是失敗還是成功的訊息
    // 產生彈跳視窗的HTML
    const toastHtml = `
        <div id="${toastId}" class="toast align-items-center text-white ${toastColorClass} border-0" role="alert" aria-live="assertive" aria-atomic="true">
            <div class="d-flex">
                <div class="toast-body">${msg}</div>
                <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast"  aria-label="Close"></button>
            </div>
        </div>
    `;

    // 把彈跳視窗的HTML插入到頁面中
    const toastContainer = document.getElementById('toastContainer');
    toastContainer.insertAdjacentHTML('beforeend', toastHtml);

    // 顯示彈跳視窗
    const toastElement = $(`#${toastId}`);
    toastElement.toast('show');

    // 當彈跳視窗關閉時，移除彈跳視窗的HTML
    toastElement.on('hidden.bs.toast', function () {
       toastElement.remove();
    });

    // 2秒後，自動關閉彈跳視窗
    setTimeout(() => {
       toastElement.toast('hide');
    }, 2000);
}

```

這樣前端就可以接收到後端傳送過來的訊息了

---

以上就是這個專案的第一個階段

這個階段的目標是要建立一個可以控制 Docker 的後台

並且透過 WebSocket 來即時更新狀態


## 結論

---

我們回顧一下這篇文章

專案最初使用了 Django 來建立一個簡單的後端

然後，我們透過 Docker SDK 來實現控制 Docker

又透過 Django RQ 來實作 Task Queue 來非同步處理 Docker 指令

最後，透過 Django Channels 來實作 WebSocket Server

前端則是使用一些簡易頁面來展示，並且以原生的 WebSocket 來接收訊息



這個專案受到下面這個專案很大的啓發

> [MahmoudAlyy/docker-django-ui](https://github.com/MahmoudAlyy/docker-django-ui)

我把它重構並改寫了一個新的版本

但爲了達成上一篇文章所說的目標

**『以Django爲後端開發集成式管理的Docker開發環境』**

我們還需要一些努力！下一篇文章終於可以來介紹第二個階段的專案了！

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E4%BB%A5django%E5%BB%BA%E7%AB%8Bdocker-gui%E6%8E%A7%E5%88%B6%E7%95%8C%E9%9D%A2-357a9987e8f3)


