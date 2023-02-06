---
title: 在 Raspberry Pi Zero 2 W上安裝 64-bit 的 Pi OS
categories: dev
tags:
  - raspberry
  - pi
  - linux
  - os
abbrlink: 6ac02cd9
date: 2023-01-25 00:00:00
---

![rpi-zero](https://i.imgur.com/MbYl55m.jpg)

## 前言

不久前搞了一台 Raspberry Pi Zero 2 W，最近想說把裏面的 OS 換成 64-bit。

---

<!--more-->

## 內容

封面圖照的灰塵有點多（懶得清理）

話說回來，之前有一篇文章在講 Raspberry Pi 4B 用 SSD 裝 64-bit OS 的文章

可以參考下面的連結：

- [Blog 文章連結](https://natlee.github.io/Blog/posts/21d73346/)
- [Medium 文章連結](https://medium.com/@natlee_/%E5%9C%A8ssd%E4%B8%8A%E7%88%B2raspberry-pi-4b%E5%AE%89%E8%A3%9D64-bit%E7%9A%84%E4%BD%9C%E6%A5%AD%E7%B3%BB%E7%B5%B1-4cd7f93799f3)

這次主要是爲 Raspberry Pi Zero 2 W 裝上 64-bit 的 Pi OS，因爲想要讓它看起來小顆一點

儲存裝置就只 SD 卡來裝 OS 了！

原本是想用同樣 S 牌的 32GB 灰白卡頂一下，但因爲 32GB 的容量有點不夠

因此去某購物網站搞了一個 Sandisk 的 128GB 的 SD 卡

![SD卡頁面](https://i.imgur.com/NEKHI6z.png)

這邊要驚歎一下，過年時間還有物流真是太可怕了

順道一提，有關 2023 年在樹莓派上的 SD 卡的比較可以參考這邊：

- [Best microSD Cards for Raspberry Pi 2023](https://www.tomshardware.com/best-picks/raspberry-pi-microsd-cards)

因爲特價，我還順便買了另一個 256GB 的做別的用途 XD

![SD卡](https://i.imgur.com/XJcDJJ8.jpg)

SD 卡到手之後，把兩個放在一起比較，`Extreme Pro`的質感看起來果然比較好（廢話）

不過 Pro 的版本會多送一個讀卡的 Adapter，但現在很多讀卡機都可以直接讀小卡了

接著重頭戲來了，我們要把 OS 燒進 SD 卡中

### 下載樹莓派鏡像燒錄軟體

現在裝 Pi OS 都有圖形化界面了，以前還要用[dd 指令](https://blog.gtwang.org/linux/dd-command-examples/)去把鏡像燒到隨身碟 XD

我們直接前往[Raspberry Pi OS 下載點](https://www.raspberrypi.com/software/)

![index](https://i.imgur.com/xSjJj13.png)

看到這個畫面直接點選 Download 就行了（用什麼作業系統去燒應該都沒問題）

![sw](https://i.imgur.com/mqqanbc.png)

安裝完點開會看到這個畫面，我們就可以直接開選了

1. 選擇 Raspberry Pi OS (other)

   ![step1](https://i.imgur.com/pc4JTIW.png)

2. 選擇 Raspberry Pi OS (64-bit)

   ![step2](https://i.imgur.com/xqfTDQ7.png)

3. 選擇你的 SD 卡後，沒意外會跳出選項問你要不要先前置設定一點東西

   這邊建議選擇`永久保存`，避免之後還要再調東調西

   ![step3](https://i.imgur.com/4sUdZUt.png)

   基本上就是照著你的需求輸入，例如：

   - 設定主機名稱（建議先設定）
   - 要不要啓用 SSH（建議先設定）
   - 自定使用者名稱密碼（建議先設定）
   - 先設定 WIFI 連線（建議先設定）
   - 等等

   這邊弄完後，按下燒錄鍵等待完成即可

接著，把燒完的 SD 卡插入到 pi 上面就可以開機了！

如果剛剛有設定 WIFI 連線的話，我們理論上可以直接在路由器找到它

或是有設定主機名稱的話，我們在同個網段就可以直接使用 SSH 連過去

例如：

```bash
ssh <你的PI上使用者名稱>@rpi.local
```

### 安裝完的調整

我們用 SSH 連上去之後，可以使用`sudo raspi-config`去調整一些設定

![config](https://i.imgur.com/1khoRH0.png)

例如進入`System Options`把自動登入功能關掉

或是，進到`Interface Options`開啓 VNC，這樣我們就可以用 GUI 去進行操作

然後關於如何把畫面調炫砲一點，可以參考下面這個影片：

[![How I Theme My Raspberry Pi OS](http://img.youtube.com/vi/gHUjO6MK5fg/0.jpg)](http://www.youtube.com/watch?v=gHUjO6MK5fg "How I Theme My Raspberry Pi OS")

簡單來說，就是安裝一些主題如下：

```bash
sudo apt install arc-theme papirus-icon-theme breeze-cursor-theme -y -qq
```

然後執行調整主題的 GUI，進去裏面設定

```bash
lxappearance
```

### SD Card 速度測試

最後，我們不免俗還是來測試一下 SD 卡的速度

樹莓派 OS 本身就有內建診斷軟體

![diag](https://i.imgur.com/HzyQyiy.png)

從開始那邊就可以找到這個軟體，叫做`Raspberry Pi Diagnostics`

![test](https://i.imgur.com/TSH2C3w.png)

點下去之後，會開啓這個視窗問你要不要測試 SD 卡的速度，當然是`Run Tests`！

測試完成後，它就會吐出如下面這樣的資料：

```
Raspberry Pi Diagnostics - version 0.12
Wed Jan 25 22:16:16 2023

Test : SD Card Speed Test
Run 1
prepare-file;0;0;20858;40
seq-write;0;0;20270;39
rand-4k-write;0;0;3844;961
rand-4k-read;7718;1929;0;0
Sequential write speed 20270 KB/sec (target 10000) - PASS
Random write speed 961 IOPS (target 500) - PASS
Random read speed 1929 IOPS (target 1500) - PASS
Test PASS
```

我們可以看到`Sequential write speed`大約是`20270 KB/sec`

另外，我們也可以直接用`dd`指令去測試讀取速度，如下

```bash
dd if=/dev/zero of=./speedTestFile bs=20M count=5 oflag=direct
```

```
5+0 records in
5+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 4.79558 s, 21.9 MB/s
```

得到的結果都差不多是 20MB/s 上下

不過畢竟是 Zero Pi，所以也不意外速度遠遠達不到 SD 卡標榜的 A2 水平（200MB/s）

另外，我有注意到在 Tom's Hardware 的網站有提到每張 SD 卡的 1024K 讀寫速度比較

![tomhardware](https://i.imgur.com/Cfj8JCA.png)

上面所說的卡都是 A1 規格，可以發現我用 A2 的 S 牌 Extreme Pro 測出來的結果跟上面同型號的 A1 規格來比

速度差不多都是 20MB/s 上下，也就是說在 Zero Pi 2 W 上用 A1 的 SD 卡就很夠了

### 同場加映

有人可能有興趣爲什麼可以當 USB 直接插

因爲我有另外去 Amazon 買[iUniker USB 擴展套件](https://www.amazon.com/dp/B07NKNBZYG?pd_rd_w=HS0EU)

這個就是它把 micro USB 的 pin 再拉出來（不用焊接）變 USB

![](https://i.imgur.com/et1m3E8.png)

然後剛剛那個套件只照顧到背面，表面可能會有板子露出，所以再買這個[iUniker 殼](https://www.amazon.com/dp/B075FLGWJL)

![](https://i.imgur.com/LRk5nfJ.png)

這樣組裝起來整個就都包起來了 😊

## 結論

現在幫樹莓派安裝 OS 的方式已經變得非常簡單，不過樹莓派現在的問題在於價格越來越高了 😥

我也算盤一波直接買 A2 的 SD 卡，但現在要找 128GB 的 SD 卡大多都是 A2 規格 😫

想說這張卡哪天或許可以挪作他用就趁特價買了

---

## Reference

- [Installing Raspberry Pi OS on a Zero 2 W](https://virtualizationreview.com/articles/2022/02/17/install-raspberry-pi-os.aspx)

- [How to Test Speed of Raspberry Pi SD Card](https://linuxhint.com/raspberry-pi-sd-card-speed-test/#:~:text=To%20open%20this%20tool%2C%20head,the%20%E2%80%9CRun%20Tests%E2%80%9D%20button.)

- [Best microSD Cards for Raspberry Pi 2023](https://www.tomshardware.com/best-picks/raspberry-pi-microsd-cards)

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E5%9C%A8-raspberry-pi-zero-2-w%E4%B8%8A%E5%AE%89%E8%A3%9D-64-bit-%E7%9A%84-pi-os-813a26d306f1)
