---
title: BlueStacks 在有 Hyper-V 的 Windows 11 上的支援狀況後續
categories: dev
tags:
  - bluestacks
  - nox
  - ldplayer
  - windows
  - hyperv
abbrlink: cc3cb59a
date: 2023-11-09 00:00:00
---

![](https://cdn-www.bluestacks.com/bs-images/Bluestacks-vs-competitors_Main-Element-1-11.png)

## 前言

今年三月的時候，我發了一篇關於各大模擬器[在有Hyper-V的Windows上BlueStacks使用狀況](https://medium.com/@natlee_/bluestacks-%E5%8F%8A-nox-player-%E5%9C%A8%E6%9C%89-hyper-v-%E7%9A%84-windows-11-%E4%B8%8A%E7%9A%84%E6%94%AF%E6%8F%B4%E7%8B%80%E6%B3%81-9a8c2ca8f13)的文章

其中有提到BlueStacks在有Hyper-V的Windows上無法使用

但在2023年的11月，我竟然看見了曙光

---

<!--more-->

## 內容

先說，這篇文章也不是業配，應該也沒人能夠同時接同個領域各種競爭對手的業配吧🤣

因爲無法同時使用Hyper-V及BlueStacks的狀況，我安裝了兩款Android模擬器

- 雷電模擬器 LD Player
- 夜神模擬器 NOX

兩個模擬器分別安裝了不同的遊戲，LD Player安裝了公主連結 Re:Dive，NOX Player安裝了Blue Archive
問題來了，爲什麼要安裝兩個模擬器呢？

因爲Blue Archive在LD Player上會卡住，而在NOX Player上可以正常運行

但這兩款模擬器都有一個共同的問題，就是速度比起BlueStacks來說慢很多

而且它們一直彈出的廣告也很煩人（螢幕右下方不定時跳出）

所以我決定再試試看BlueStacks

結果我沒搜尋還好，一搜尋就發現了BlueStacks的官方網站上有一篇文章：

[BlueStacks vs LD Player vs Memu vs NOX](https://www.bluestacks.com/tw/bluestacks-vs-ldplayer-vs-memu-vs-nox.html)

這篇文章裏面有各種模擬器的比較，我這邊就不多說了，直接貼它們的圖表

### 測試環境

![](https://i.imgur.com/cAj2ad5.png)

我個人認爲這個測試環境的重點是`虛擬化開啟`

就跟原神啓動一樣，如果沒有開啓虛擬化，那就跟上篇文章一樣，BlueStacks是可以直接下去的

### 各種比較結果

#### CPU使用量

![](https://i.imgur.com/8MjancG.png)

其實一直以來，我都覺得BlueStacks的CPU使用量很低

但是我直接觀察工作管理上其他模擬器的CPU使用量，也沒有比BlueStacks高多少

可能是體感問題，但基於這個圖表，BlueStacks的CPU使用量確實是最低的

#### APP啓動速度

![](https://i.imgur.com/I119q3p.png)

關於App的啓動速度，這個圖表的結果跟我個人的體感是一致的

真的要不是因爲不能同時使用虛擬化，我也不會放棄BlueStacks

#### 單一多開記憶體使用量

![](https://i.imgur.com/6ctm5Ap.png)

單一多開的狀況，我體感上覺得每個模擬器的記憶體使用量都差不多

不過BlueStacks與用量最多的夜神模擬器最多也差不到兩倍，所以還算是可以接受的範圍

但夜神模擬器不但記憶體用量高，而且整體的速度還超級慢的

要不是因爲Blue Archive在LD Player上卡住，我也不會去用夜神模擬器

#### 多個多開記憶體使用量

![](https://i.imgur.com/3I30r9V.png)

我只有在夜神模擬器上開了一個instance，所以沒有感受到這個問題

不過光看圖表上，夜神開了三個instance的時候，等於它本身開一個的時候的三倍

以程式面來說，直接乘以三倍的記憶體使用量，代表它的程式碼完全沒有考慮到多開的情況

這種結果來看，我覺得夜神模擬器需要在多開上面多花一點功夫

#### APK安裝時間

![](https://i.imgur.com/p0u1hH9.png)

APK通常都是一次性的，所以我覺得安裝速度可能沒有那麼重要

但一般使用者對模擬器的第一印象，可能就是安裝速度

所以BlueStacks特別去做了這個測試，我覺得是真的有考慮到使用者的感受

#### Geekbench跑分

![](https://i.imgur.com/L0xPdSy.png)

基本上，不只是模擬器，很多軟體都會針對跑分去做優化

我個人認爲，跑分只是一個規格的參考

除分真的差太多，不然不是最重要的，使用者體驗遠比跑分來得更重要

### 結語

這個頁面的最後，有一題常見問題及回答

![](https://i.imgur.com/5cNpIsz.png)

不知道其他人怎麼看這個說法

我覺得BlueStacks算是很老牌的模擬器了，所以市佔率也應該要是最高的

總之，很慶幸我能夠在今年結束前看到BlueStacks這次的更新

感謝BlueStacks的開發者們，讓我能夠在Windows 11上搭配虛擬環境使用BlueStacks

另外，這篇文章沒有要吹捧BlueStacks，畢竟軟體開發這條路上，沒有誰是完美的

過去半年，雷電模擬器在當時的環境下，是我最佳的選擇

但現在BlueStacks也可以在有Hyper-V的環境下使用了

代表說，BlueStacks的開發者們也在持續優化這個產品

對於這種能夠看見使用者需求的公司，我還是很願意支持的

套一句上一篇文章的話

> 誰先早一步做完必要的功能就能搶到更多使用者

這句的下一句是『就算晚一步，只要做得更好，還是能夠搶回來』

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/bluestacks-%E5%9C%A8%E6%9C%89-hyper-v-%E7%9A%84-windows-11-%E4%B8%8A%E7%9A%84%E6%94%AF%E6%8F%B4%E7%8B%80%E6%B3%81%E5%BE%8C%E7%BA%8C-0dc4e2201738)


