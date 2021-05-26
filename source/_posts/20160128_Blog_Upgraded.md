title: Blog近代化改修
date: 2016-01-28
categories: Blog
tags: [Blog]
---

![何等鬼畜的行為](http://i.imgur.com/x3aK407.jpg)
<center>*為美好的世界獻上祝福*</center>


前言
--
 ----------

趁還有一點點時間來清裡長滿蜘蛛網的Blog。
然後加一些功能，順便記錄一下以免以後忘記。

<!--more-->

## 近代化改修 ##
----------


### 增加右下多功能鍵 ###

> **<font color="red">※ 此篇是基於nexT主題的改修</font>**

首先抓住**藏在themes/next/layout底下的小傢伙**

> _layout.swig


摸了一下發現這玩意掌管整個Blog的布局
幾乎每個頁面都靠它插html進去

然後找到footer裡面跟原生theme界面很要好的那個置頂功能物件
先把原本的功能註解掉

    <!--<div class="back-to-top"></div>-->

接下來把我們要的東西加上去

    <div id='ukagaka_panel'></div>

不過這樣還不夠，再來找到body底端把重要的心臟**css跟javascript**放上去
```js
    <script src='http://code.jquery.com/ui/jquery-ui-git.js'></script>
    <link type="text/css" rel="stylesheet" href="/Blog/css/ukagaka/ukagaka.css">
    <link type="text/css" rel="stylesheet" href="/Blog/css/ukagaka/simpleFontawesome.css">
    <script src="/Blog/js/ukagaka/jquery.morris.ukagaka.resource.js"></script>
    <script src="/Blog/js/ukagaka/typed.js"></script>
    <script type="text/javascript">
      $('#ukagaka_panel').ukagaka();
    </script>
```

> 這邊不放jquery是因為**venders.swig**裡面已經有了
> `jquery_lazyload/jquery.lazyload.js?v=1.9.7`
> 如果放了版本不相容會造成方法衝突

最後把上面所需的js跟css放到網頁底下就大功告成了


----------


### 增加404頁面 ###

> 不過實用度高嗎？

![這問題有回答的必要嗎](http://i.imgur.com/QAnIoWi.jpg)
<center>*夢幻之星2*</center>


> 首先直接在站點的source下新增一個**404.html**
> 首先直接在站點的source下新增一個**404.html**
> 首先直接在站點的source下新增一個**404.html**

這很重要，一定要講三次
再按照自己所需去修改css跟在網站底下放置所需的檔案
似乎這樣就完成了

我Github的Blog不是在帳號的最上層網域
所以一開始卡在不知道404頁面要放在哪裡

{% blockquote Github https://help.github.com/articles/custom-404-pages/ Custom 404 Pages %}
Create a new HTML file named `404.html`—or a new Markdown file named `404.md`, with `permalink: /404.html` in its YAML front matter— at the root level of your GitHub Pages repository.
{% endblockquote %}

看了官方文件也沒看懂哪個方法才要在root level上面
然後查了網上也滿多人說要在root level才能自己建404

![別給我滿嘴開火車](http://i.imgur.com/fGXnfga.jpg)
<center>*夢幻之星2*</center>

原本準備要放棄洗洗睡，但不知為何又燃起一種再試試看的決心
試了一段時間發現，果然真的要講三次
真的只要在站點的source下新增一個**404.html**就行了

[首頁的404](http://natlee.github.io/ERROR)
[Blog的404](http://natlee.github.io/Blog/ERROR)

終於弄完才發現這個404的功用好像也只有現在打打文章可以用到
一般瀏覽Blog的時候也不會特別發生或去點到404...
但還是補充一下，雖然都是直接加**404.html**
Hexo的404會自動嵌到Blog的網頁框框內
我覺得這樣的確是還滿不錯的

![這裡也有一個麻煩的傢伙啊](http://i.imgur.com/66iLHqa.jpg)
<center>*為美好的世界獻上祝福*</center>


----------

## 總結 ##

最後，這邊推荐兩個不錯又實用的網站

[Html5up](http://html5up.net/)
[Wibibi](http://www.wibibi.com/)

第一個裡面有現成的HTML5素材可以拿來參考或使用
第二個裡面有很多CSS的範例也是可以拿來參考或使用


總之Blog更新先到這邊
以後有時間再來研究一下

對了，覺得404畫面的人物滿可愛的話(?)
可以這邊請

[Line貼圖 - Lovely Blond Hair Girl](http://line.me/S/sticker/1226938)

謝謝大家，祝 **新年快樂**
