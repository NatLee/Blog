---
title: 在 Django 中使用 Allauth 綁定 JWT 登入
categories: dev
tags:
  - python
  - django
  - allauth
  - jwt
abbrlink: 65209cf0
date: 2024-08-11 00:00:00
---

![](https://i.imgur.com/4o87A4H.png)

## 前言

---

明明前後端分離這麼流行，為什麼 Django 的 Allauth 套件沒有支援直接返回 JWT 的功能呢？
這篇文章就來探討一下其中的奧妙之處

<!--more-->

## 內容

---

眾所周知，Django 的 Allauth 套件是一個非常強大的第三方登入套件，可以讓我們快速的實現第三方登入的功能
但是它的登入方式是基於 session 的，這對於前後端分離的專案來說，就有點不太方便了

### 前後端分離的 Google 登入

一般來說，前後端分離的專案通常是使用 JWT 來做登入驗證，這邊使用 Django 以 Google 登入為例，來看看前後端如何串接

#### 前端

前端在HTML的部分，我們需要引入 Google 登入的 SDK，並且設定好 `client_id` 和 `callback`

其中，Google的程式碼會在前端動態載入，去判斷`client_id`跟`當前網址`是不是可以被信任
也就是說，GCP專案憑證上設定的`callback`必須跟前端設定的`callback`一致，否則會出現錯誤

```html
<meta name="google-signin-scope" content="profile email">
<meta name="google-signin-client_id" content="{{ client_id }}">
<script src="https://accounts.google.com/gsi/client" async defer></script>
<div id="g_id_onload" data-client_id="{{ client_id }}" data-callback="getJWTUsingGoogleCredential"></div>
```

當使用者登入成功後，會呼叫 `getJWTUsingGoogleCredential` 這個函數，將 Google 登入的 `credential` 傳送給後端

```javascript
function getJWTUsingGoogleCredential(data) {
    const credential = data.credential;
    $.ajax({
        method: "POST",
        url: "/api/auth/google/token/session",
        data: { credential: credential }
    }).done(function (data) {
        const access_token = data.access;
        const refresh_token = data.refresh_token;
        localStorage.setItem('access_token', access_token);
        localStorage.setItem('refresh_token', refresh_token);
        /* ... */
    });
}
```

#### 後端

在後端的部分，我們需要自定義一個 Serializer 來處理 Google 登入的 `credential`，並且驗證這個 `credential` 是否有效

```python
from google.oauth2 import id_token
from google.auth.transport import requests

class GoogleLoginSerializer(serializers.Serializer):
    # Google login
    # 用於接收Google返回的憑證
    credential = serializers.CharField(required=True)
    def verify_token(self, credential: str) -> dict:
        """
        驗證Google返回的id_token
        credential: JWT格式的字串
        """
        # 使用Google提供的方法驗證token
        idinfo = id_token.verify_oauth2_token(
            credential,
            requests.Request(),
            settings.SOCIAL_GOOGLE_CLIENT_ID
        )
        # 驗證token的發行者
        if idinfo["iss"] not in [
            "accounts.google.com",
            "https://accounts.google.com",
        ]:
            logger.error("Wrong issuer")
            raise ValueError("Wrong issuer.")
        # 驗證token的受眾
        if idinfo["aud"] not in [settings.SOCIAL_GOOGLE_CLIENT_ID]:
            logger.error("Could not verify audience")
            raise ValueError("Could not verify audience.")
        # 驗證成功
        logger.info("successfully verified")
        return idinfo
    # ...
```

裡面會拿前端傳過來的 `credential` 去跟Google驗證，並且返回一個 `idinfo`
這個 `idinfo` 就是 Google 登入的資訊，裡面包含了使用者的名字及電子郵件資訊

其中， `SOCIAL_GOOGLE_CLIENT_ID` 是我們在 GCP 專案中設定的 `client_id`，這邊要注意的是，這個 `client_id` 必須跟前端設定的 `client_id` 一致

`idinfo`的範例資料如下：

```json
{
    // These six fields are included in all Google ID Tokens.
    "iss": "https://accounts.google.com",
    "sub": "110169484474386276334",
    "azp": "1008719970978-hb24n2dstb40o45d4feuo2ukqmcc6381.apps.googleusercontent.com",
    "aud": "1008719970978-hb24n2dstb40o45d4feuo2ukqmcc6381.apps.googleusercontent.com",
    "iat": "1433978353",
    "exp": "1433981953",

    // These seven fields are only included when the user has granted the "profile" and
    // "email" OAuth scopes to the application.
    "email": "testuser@gmail.com",
    "email_verified": "true",
    "name" : "Test User",
    "picture": "https://lh4.googleusercontent.com/-kYgzyAWpZzJ/ABCDEFGHI/AAAJKLMNOP/tIXL9Ir44LE/s99-c/photo.jpg",
    "given_name": "Test",
    "family_name": "User",
    "locale": "en"
}
```

其他詳細的資料可以參考 [Google 官方文件](https://developers.google.com/identity/sign-in/web/backend-auth?hl=zh-tw)

當驗證成功後，我們就可以使用 `idinfo` 中的資料來創建或更新使用者的資料，並且返回 JWT 給前端

但是問題來了，不是所有的第三方登入都像 Google 一樣提供了前端動態載入的一頁式 SDK，那該怎麼辦呢？

而且只有登入的部分是不夠的，我們還需要處理登出、綁定、解綁等等的功能！

### Allauth 與 JWT

如果我們一個個去實現所有的第三方登入，那就太麻煩了，這時候 Allauth 就派上用場了

根據[Allauth官方文件](https://docs.allauth.org/en/latest/installation/quickstart.html)，我們可以快速的將他整合到我們的專案中

他有非常多的providers可以使用，包括 Google、Facebook、Twitter、Github等等

但要記得把我們需要的部分加入到 `INSTALLED_APPS` 中

```python
INSTALLED_APPS = [
    # ...
    # ====================
    # Allauth Apps
    # ===================
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.google',
    # ...
]
```

在URL的部分，不同的provider會有不同的路徑，當我們將他的URL加入到我們的專案中後，就可以直接使用他的登入功能

假設我們在專案加入一個 app ，叫做 `authentication`，我們可以在專案以及app的 `urls.py` 中加入以下的路徑：

```python
# project/urls.py
urlpatterns = [
    # ...
    # ===================
    # Custom Allauth Routes
    # ===================
    path('api/allauth/', include('authentication.urls')),
    # ...
]

# authentication/urls.py
urlpatterns = [
    # ===================
    # Django Allauth
    # ===================
    # Include Allauth Routes
    path('', include('allauth.urls')),
]

```

當我們已經綁定好路徑後，我們就可以直接使用 `allauth` 提供的登入功能了

```html
<!-- 第三方登入功能 -->
<a title="Google" href="/api/allauth/google/login/?process=login" class="social-login-btn">登入 Google</a>
<!-- 第三方綁定功能 -->
<a title="Google" href="/api/allauth/google/login/?process=connect" class="social-login-btn">綁定 Google</a>
```

不過，我們如果直接點下登入的話，會發現他是基於 session 的登入的
最後，跳轉回來的時候，會發現他是直接返回到 `/accounts/profile/`
這樣就無法直接返回 JWT 給前端了！

因此，我們需要自定義一個 `Adapter` 來處理複寫 `allauth` 的登入流程

我們先在 `authentication` app 中新增一個 `adapters.py`，並且繼承 `DefaultSocialAccountAdapter`

然後，我們可以在 `get_connect_redirect_url` 這個函數中處理社交帳號連接的重定向

```python
from allauth.socialaccount.adapter import DefaultSocialAccountAdapter

class MySocialAccountAdapter(DefaultSocialAccountAdapter):
    def get_connect_redirect_url(self, request, socialaccount):
        # 處理社交帳號連接的重定向
        base_url = reverse('social-connect-callback')
        return base_url
```

接著，我們在 `authentication` app 中新增一個 `views.py`以及一個 `urls.py`，來處理社交帳號的連接

```python
# authentication/views.py
from django.shortcuts import render

def social_login_callback(request):
    return render(request, 'social_login_callback.html')

# authentication/urls.py
urlpatterns += [
    # ===================
    # Custom Callback Route
    # ===================
    # 第三方帳號登入 Callback
    path('social/login/callback', views.social_login_callback, name='social-login-callback'),
]

```

最後，我們建立一個空白檔案 `authentication/templates/social_login_callback.html`

這樣，當我們點擊登入的時候，就會跳轉到 `social_login_callback.html` 這個頁面
雖然這個頁面是空白的，但是我們可以在這個頁面中加入一些 JavaScript 來處理JWT的轉移！

我們再回到 `adapter.py` 中，我們可以再建立一個新的 `Adapter` 來處理登入的重定向

這邊我們建立一個叫做 `MyAccountAdapter` 的 `Adapter`，並且繼承 `DefaultAccountAdapter`

裡面定義了一個 `get_login_redirect_url` 函數，判斷是否為社交帳號登入，如果是的話，就返回 JWT 給前端；如果不是的話，就使用預設的重定向

```python
class MyAccountAdapter(DefaultAccountAdapter):
    def get_login_redirect_url(self, request):
        # 檢查是否為社交帳號登入
        auth_methods = request.session.get('account_authentication_methods')
        # ========================
        # 這邊判斷是否為社交帳號登入
        if auth_methods and auth_methods[0].get('method') == "socialaccount":
            # 獲取 tokens
            tokens = self.get_tokens(request)
            base_url = reverse('social-login-callback')
            query_params = urlencode({'tokens': json.dumps(tokens)})
            return f"{base_url}?{query_params}"
        # ========================
        # 如果不是社交帳號登入，使用預設跳轉
        return super().get_login_redirect_url(request)

    def get_tokens(self, request):
        from rest_framework_simplejwt.tokens import RefreshToken
        user = request.user
        refresh = RefreshToken.for_user(user)
        # 返回 tokens
        return {
            'access_token': str(refresh.access_token),
            'refresh_token': str(refresh)
        }
```

這樣我們第三方登入完成後，就會跳轉到 `social_login_callback.html` 這個頁面，並且返回 JWT 到網址上

我們只要在 `social_login_callback.html` 中去解析網址上的 JWT，並且存儲到 localStorage 中，就可以完成整個登入流程了

```html
<script>
    (function() {
        // 解析 URL 參數
        function getUrlParameter(name) {
            name = name.replace(/[\[]/, '\\[').replace(/[\]]/, '\\]');
            var regex = new RegExp('[\\?&]' + name + '=([^&#]*)');
            var results = regex.exec(location.search);
            return results === null ? '' : decodeURIComponent(results[1].replace(/\+/g, ' '));
        }

        // 獲取 tokens
        var tokensJson = getUrlParameter('tokens');
        // 如果 tokens 存在，則解析並存儲到 localStorage
        if (tokensJson) {
            try {
                var tokens = JSON.parse(tokensJson);
                localStorage.setItem('access_token', tokens.access_token);
                localStorage.setItem('refresh_token', tokens.refresh_token);
                console.log('Tokens 已成功儲存');
            } catch (e) {
                console.error('解析或存儲 tokens 時發生錯誤:', e);
            }
        } else {
            console.warn('未找到 tokens');
        }
    })();
</script>
```

這樣，我們就完成了整個 Allauth 與 JWT 的串接了！

從前端點擊登入，到跳轉到我們定義的頁面，頁面的 JavaScript 會解析網址上的 JWT，並且存儲到 localStorage 中！

因為 Allauth 本身已經提供很多provider，所以我們改寫 `Adapter` 的部分就可以應對所有的第三方登入了！

最後，要記得 `project/settings.py` 中加入我們自定義的 `Adapter`：

```python
ACCOUNT_ADAPTER = 'authentication.adapters.MyAccountAdapter'
SOCIALACCOUNT_ADAPTER = 'authentication.adapters.MySocialAccountAdapter'
```

## 結論

---

這篇文章想分享在 Django 中使用 Allauth 綁定 JWT 登入的方法

但除了文章內改寫的部分外，還有很多眉角，例如：

- 帳號的email是否重複
- 是否有重複綁定的問題
- Email的 domain 是否屬於白名單
- Callback視窗是否會自己關閉
- 主視窗要怎麼知道登入成功
- 等等...

這些需要考慮的問題有在我GitHub上專案中的程式碼有提供解決方案：

- [django-allauth-JWT-template/authentication/adapter.py#L45](https://github.com/NatLee/django-allauth-JWT-template/blob/main/backend/authentication/adapter.py#L45)

這個專案有提供一個playground，可以直接測試各種功能：

![allauth-jwt-template](https://github.com/NatLee/django-allauth-JWT-template/raw/main/doc/dashboard.png)

原本我不想串接Allauth，因為在前後端分離的狀況下，session登入有點難搞

但是後來發現Allauth提供的provider非常多，比起自己一個個維護

只要多一個callback頁面，就可以省下一堆麻煩，那還有不用的理由嗎？

另外，同場加映的是，我有做一個Django Template的專案，裡面有很多常用的功能，列出其中幾項如下：

- Containerized
- Nginx
- Async Task(Django-Q2)
- MySQL(MariaDB)
- API Proxy
- User Login History
- WebSocket
- JWT
- 3rd Party Login
- ...

總之就是一包瑞士刀，有興趣的人可以參考一下[Django Project Template](https://github.com/NatLee/django-project-template)

裡面的儀表板看起來比較炫砲一點

![template-dashboard](https://github.com/NatLee/django-project-template/raw/main/doc/cover.png)

歡迎大家一起交流！

## Reference

---

- [Django Allauth](https://docs.allauth.org/en/latest/index.html)

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E5%9C%A8-django-%E4%B8%AD%E4%BD%BF%E7%94%A8-allauth-%E7%B6%81%E5%AE%9A-jwt-%E7%99%BB%E5%85%A5-51f6d4436f86)

