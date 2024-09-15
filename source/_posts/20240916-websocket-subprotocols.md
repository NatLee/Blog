---
title: 優化WebSocket連接：使用Subprotocol增加安全性
categories:
tags:
  - WebSocket
  - Django
  - Authentication
abbrlink: 8f2e9a31
date: 2024-09-16 00:00:00
---

![](https://i.imgur.com/3xURd7c.png)

## 前言

在現代網頁中，WebSocket連接扮演著越來越重要的角色，尤其是在需要即時通訊的場景中

---

<!--more-->

## 內容

這邊不贅述WebSocket的基本原理，而是著重於如何在Django中實現WebSocket連接的安全性

最近，我有一個 side project [Telepy](https://github.com/NatLee/telepy) 使用到了web terminal，而這個terminal是基於WebSocket連接的

在這個過程中，我發現了一個問題：WebSocket連接的安全性

### 第一次踩坑

一開始，我只是簡單地使用了Django Channels來實現WebSocket連接

但是這樣做有一個安全隱憂：連接的建立過程中，沒有任何認證機制！

這意味著任何人都可以通過WebSocket連接到我的server，這是非常危險的😮

於是我開始思考該如何在WebSocket連接中實現認證

然後我把目光轉向了URL Query String

我就快樂的把這個功能實現出來，但這個方法有一個致命的缺點！

**URL Query String是明文的！**

### 第二次踩坑

有了第一次的經驗，我又思考了一下

如果URL不行的話，是不是有方法能夠像HTTP一樣傳遞Header？

於是我找到了WebSocket Subprotocol


#### 什麼是Subprotocol？

Subprotocol是WebSocket的一個欄位，它允許使用者和伺服器在建立連接時傳遞一些參數

這些參數通常是拿來告訴伺服器接下來的傳輸要使用什麼協議

例如，我可以跟伺服器說：我要使用`JSON`協議來傳輸資料，這樣伺服器就會知道要用`JSON`來解析資料

這個功能在WebSocket連接中非常有用，而且也很安全！

因為Subprotocol在建立連接時就會被傳遞，這個傳遞過程會被WSS保護，所以不用擔心中間人攻擊

但違背這個功能的初衷，我們可以利用Subprotocol來傳遞一些認證資訊😎😎😎

### 用力更新一波

這次在專案中的更新主要集中在`TerminalConsumer`中

裡面引入了WebSocket Subprotocol的使用，這個改變主要影響了連接的建立過程，特別是在認證和參數傳遞方面


1. **引入Subprotocol**
    
    更新後的程式碼碼現在會檢查WebSocket連接請求中的Subprotocol：

    ```python
    if self.scope['subprotocols']:
         for protocol in self.scope['subprotocols']:
              if protocol.startswith('token.'):
                    base64_encoded_token = protocol.split('.', 1)[1]
                    token = base64.b64decode(base64_encoded_token).decode()
              elif protocol.startswith('server.'):
                    server_id = protocol.split('.', 1)[1]
              elif protocol.startswith('username.'):
                    username = protocol.split('.', 1)[1]
              elif protocol.startswith('auth.'):
                    subprotocol_auth = protocol
     ```

     這邊可以注意到，使用Django Channels的`self.scope['subprotocols']`可以獲取到Subprotocol的資訊

2. **參數檢查**

    在更新後的程式碼中，我們會檢查Subprotocol中的參數是否合法：

    ```python
     if not token or not server_id or not username or not subprotocol_auth:
          logger.error("Missing required subprotocols")
          await self.close(code=4000)
          return
    ```

    如果參數不合法，連接會被強制關閉

    ![](https://i.imgur.com/ParzCAg.jpeg)


3. **接受連接**

    在檢查完參數後，我們就可以在後端接受連接了：

    ```python
    await self.accept(subprotocol=subprotocol_auth)
    ```

    這邊的`subprotocol_auth`就是我們在Subprotocol中傳遞的認證資訊

    如果我們沒有在`self.accept()`中傳入`subprotocol`的話，連接會被拒絕

    ![](https://i.imgur.com/8OHIot0.png)

    因為對於`subprotocol`來說，這本來就是跟後端溝通用的`協議`！


4. **前端的更新**

    在前端，將原先的WebSocket連接的建立過程：

    ```javascript
    const ws = new WebSocket(`${ws_scheme}://${window.location.host}/ws/terminal/?token=${token}&server_id=${server_id}&username=${username}`);
    ```

    問題是這樣的連接方式是不安全的，其中`token`會被暴露在URL中！

    因此，我們需要將這些資訊放到Subprotocol中：

    ```js
    // 連接不再使用URL Query String
    const ws_path = `${ws_scheme}://${window.location.host}/ws/terminal/`;

    // 將需要的資訊放到Subprotocol中
    const tokenInfo = `token.${btoa(accessToken)}`;
    const serverInfo = `server.${serverID}`;
    const usernameInfo = `username.${username}`;
    // 這邊的ticket是一個自定義的token，用來做subprotocol的協議名稱
    const ticket = `auth.${btoa(`${serverID}.${username}`)}`;
    const socket = new WebSocket(ws_path, [tokenInfo, serverInfo, usernameInfo, ticket]);
    ```

    如此一來，就解決了URL Query String的問題，也增加了連接的安全性

---

## 結語

這次在side project的更新學習了如何使用Subprotocol來增加WebSocket連接的安全性

這個方法非常適合在需要認證的場景中使用，並且也很容易實現

要注意的地方是，Subprotocol的使用需要前後端的配合，因此在更新時要注意前後端的一致性

而且欄位的長度也有限制，因此在使用時要注意欄位的長度！

不能直接把整個JWT塞進去，這樣會導致Subprotocol的長度過長😂

---

## 參考資料

- [Django Channels](https://channels.readthedocs.io/en/latest/)
- [WebSocket Subprotocol](https://datatracker.ietf.org/doc/html/rfc6455#section-1.9)

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E5%84%AA%E5%8C%96websocket%E9%80%A3%E6%8E%A5-%E4%BD%BF%E7%94%A8subprotocol%E5%A2%9E%E5%8A%A0%E5%AE%89%E5%85%A8%E6%80%A7-b6b1dfb63592)

