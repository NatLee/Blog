---
title: 以 Google 表單做資料庫使用方法
date: 2017-06-08
categories: blog
tags: [google, database, googlesheet, blog]
---

![我很感動](http://i.imgur.com/CHxfjRG.jpg)
<center>*劍風傳奇*</center>

## 前言
----------

之前研究了一陣子，做出了點東西。

但記憶不佳怕忘記，還是記錄一下，順便幫助別人。

內容大概是用 Google 表單來當資料庫就可以做點事。（*這句不是跟標題一樣嗎？*）

<!--more-->

## 新增表單與 EntryID
----------
![01](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/01.png)

首先，先建立新的表單，並填好表格名稱及各種問題，題目的名字在爬蟲時會用到，故可取簡單點，像這邊取*store_data_base* 及*store_data_base_user*

![02](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/02.png)


按回覆，再按綠綠的，可以建立google表格（*建議跟名字相同，怕你像我一樣容易失憶搞混*），先建等等用。

![03](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/03.png)

按下傳送，會看到你的表單網址。Go！到你的表單去！

  1. 按下F12，找到你欄位的ID，大概會長這樣 `entry.888061137`

  2. 如有兩個(含)以上欄位可繼續，像我找到第二個是 `entry.662791979`


## 表格探險

------


![04](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/04.png)


再來進到你的google表格（可從google雲端或一開始編輯表單回覆按下綠綠的進去）

![05](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/05.png)

按下 *檔案* ，再按下 *發佈到網路* ，會看到這般光景

![06](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/06.png)

勇敢按下 *發佈* ，會取得你的表格在網路上的位址，簡稱「*網址*」

## 取得 JSON

------



![07](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/07.png)

放大一看，有沒有一串像是這樣的東西（*聽說它叫google spreadsheet key*）

※ 注意，新版Google表單在這邊會看到的是 *.../d/e/...* ，但我們要的是 *.../d/<這才是我們要的>*，那麼瀏覽器最上方網址列的網址就是我們要的。 2020-01-10

> `1cyaKtRh8o-Ngq63KWSkPTKOr_nf2ZhN0TTzTVrqpKZI`

![08](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/08.png)

接著我們要串接一個網址：

> `https://spreadsheets.google.com/feeds/worksheets/這邊放你剛剛拿到的一串東西/private/full`

我串出來長這樣，但一定跟你不一樣：

> `https://spreadsheets.google.com/feeds/worksheets/1cyaKtRh8o-Ngq63KWSkPTKOr_nf2ZhN0TTzTVrqpKZI/private/full`

在瀏覽器輸入這串網址進入後，用 Ctrl + F 搜尋 `<id>` 這個Tag，於是你會發現有個 `</id>` 前面有一段字非常潮，我這邊的潮字如下：

> `ouce9z`

![09](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/09.png)

把剛剛的潮字接到一個網址中，格式如下：

> `https://spreadsheets.google.com/feeds/list/這邊放google表格的那串/潮字/public/full?alt=json`

我接出來大概像下面這樣，你一定跟我不一樣：

> `https://spreadsheets.google.com/feeds/list/1cyaKtRh8o-Ngq63KWSkPTKOr_nf2ZhN0TTzTVrqpKZI/ouce9zs/public/full?alt=json`

哇！太棒了！你就可以進到這個網頁了！於是你可以使用其他程式語言開始爬蟲！

## 利用網址 Input 新增資料

------



![10](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/10.png)

這樣就結束？還沒！回到你的表單，並填好資料按送出！

送出後看看神奇的網址，太酷啦！

觀察一下會發現，*真是擅常送出表單的朋友呢！*

在 *formResponse* 後面加上「?」再加上你的欄位 ID （*也就是一開始找的 Entry ID，如有兩個(含)以上請用&連接*），就會得到以下網址：

> `https://docs.google.com/forms/d/e/你剛剛網址中的formResponse前那串字/formResponse?你的EntryID=你的答案&你的EntryID=你的答案`

> **<font color="red">※ 避免此表單被新增故不直接貼完整網址請見諒</font>**

![11](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/11.png)

測試一下輸入 *你的答案* 在網址上並按回車鍵！回車！也就是Enter。

> 「什…什麼！騙人的吧！」
>
> 「我____都拿出來了，你給我看這個？」
>
> 「你什麼時候產生了我對你使用鏡花水月的錯覺？」

對的，網站還是同一頁。

這真是非常的感傷，直到你心灰意冷，請打開google表格。

![12](https://raw.githubusercontent.com/NatLee/BlogSource/master/Image/Google_Spreadsheet_Database/12.png)

わ―い！すご―い！這真是驚人的大發現。

於是你就可以利用 google表單 來進行「*這樣還有那樣*」的事了！

------

對了，在最後 *工商服務*  一下

新的 Line貼圖 上架了，覺得喜歡的可以幫忙支持一下

[顏肥與夥伴們](https://store.line.me/stickershop/product/1446286)

謝謝大家！



## 參考資料

------

[[Retrieve Google Spreadsheet Worksheet JSON](https://stackoverflow.com/questions/24531351/retrieve-google-spreadsheet-worksheet-json)]
