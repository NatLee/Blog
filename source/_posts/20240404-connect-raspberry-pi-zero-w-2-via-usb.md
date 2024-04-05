---
title: 使用USB Ethernet Gadget連接Raspberry Pi Zero W 2
categories: dev
tags:
  - raspberry
  - pi
  - linux
  - ethernet gadget
  - usb
  - ssh
abbrlink: db845c5f
date: 2024-04-04 00:00:00
---

![rpi-zero](https://i.imgur.com/I5WdHYc.png)

## 前言
----------

之前使用樹莓派的時候，都是透過HDMI接上螢幕，再透過鍵盤滑鼠操作。
但是萬一手邊沒有螢幕跟鍵盤滑鼠，要怎麼操作呢？

<!--more-->

## 準備開始

首先，我們需要準備以下材料：

- 1 x Raspberry Pi 4 or Zero / Zero W / Zero 2 W 或是更新
- 1 x MicroSD 卡
- 1 x 能夠讀取 MicroSD 卡的裝置
- 1 x 筆電或是桌機
- 1 x MicroUSB to USB 連接線

> 筆電或是桌機必須是`Windows`，因為`Linux`設定太坑了，所以我沒成功過，然後`MacOS`可能可以，但我沒試過

## 安裝步驟

1. 我們要先下載 Raspberry Pi OS 的映像檔並安裝，這邊建議直接使用 [Raspberry Pi Imager](https://www.raspberrypi.org/software/)
    > 這邊注意！建議選`64-bit Full`的版本，其他版本我沒有試過，不確定是否有支援
    ![select-os](https://i.imgur.com/aZJZlVE.png)

2. 設定安裝的主機名稱（hostname）以及使用者帳密（username/password）
    ![customize-settings](https://i.imgur.com/nq6Mx1m.png)

3. 等待安裝完成，並重新插入 MicroSD 卡到電腦中

4. 找到 MicroSD 卡的 `bootfs` 磁碟中的 `config.txt` 檔案

    1. 新增一行在特定位置
        > 位置一定要對！網路上很多教學都只說新增一行，但沒說要新增在哪裡🥹
        ```toml
        ...
        # Enable audio (loads snd_bcm2835)
        dtparam=audio=on
        dtoverlay=dwc2

        # Automatically load overlays for detected cameras
        ...
        ```
        結果如下：
        ![add-dtoverlay](https://i.imgur.com/X6CmYE9.png)

    2. 接著，將`otg_mode=1`給註解掉
        ```toml
        ...
        [cm4]
        # Enable host mode on the 2711 built-in XHCI USB controller.
        # This line should be removed if the legacy DWC2 controller is required
        # (e.g. for USB device mode) or if USB support is not required.
        # otg_mode=1 <---- 就是這行

        [all]
        ...
        ```
5. 找到 MicroSD 卡的 `bootfs` 磁碟中的 `cmdline.txt` 檔案

    1. 在`rootwait`後面加上`modules-load=dwc2,g_ether`
        > 因為cmdline只有一行，要注意不要加錯位置
        ```
        ... rootwait modules-load=dwc2,g_ether quiet splash ...
        ```
        會變成這樣：
        ![add-modules-load](https://i.imgur.com/3RRKhkd.png)

6. 將 MicroSD 卡退出，插入到 Raspberry Pi Zero W 2 中，並連接 MicroUSB to USB 連接線到電腦上。我是用[Raspberry Pi USB Dongle Expansion Board Kit](https://www.amazon.co.jp/-/en/Raspberry-Dongle-Expansion-GeeekPi-Included/dp/B07V3DY76F)，所以可以直接插上去
    ![insert-pi](https://i.imgur.com/rrIMM31.png)

7. 如果你的電腦是`Windows`，會需要安裝`RNDIS`驅動程式，可以參考[這篇](https://wiki.sipeed.com/hardware/en/maixsense/maixsense-a075v/install_drivers.html#Download-driver)

    安裝完成後，可以在裝置管理員中看到`RNDIS`的裝置
    ![device](https://i.imgur.com/v3W25iF.png)

    網路介面卡也會多出一個裝置，這就是直接把 Raspberry Pi Zero W 2 當成網路介面卡
    ![RNDIS](https://i.imgur.com/FO5BQva.png)

8. 接著，等待一下，Raspberry Pi Zero W 2 會自動開啟並連接到你的電腦上，這時候我們可以先ping一下看看有沒有連上
    > Pi 有可能會重新連接你的電腦，所以馬上ping可能會失敗，可以看一下網路介面有沒有連上，等一下再試試看
        ![interface](https://i.imgur.com/uLymh1F.png)

    如果網路介面沒有問題，我們可以ping一下看看有沒有連上：
    ```bash
    ping pi.local
    ```
    成功的話，你會看到類似以下的結果：
    ```
    Ping pi.local [fe80::f4da:bc9b:9cc1:2daf%26] (使用 32 位元組的資料):
    回覆自 fe80::f4da:bc9b:9cc1:2daf%26: 時間<1ms
    回覆自 fe80::f4da:bc9b:9cc1:2daf%26: 時間<1ms
    回覆自 fe80::f4da:bc9b:9cc1:2daf%26: 時間<1ms
    回覆自 fe80::f4da:bc9b:9cc1:2daf%26: 時間<1ms

    fe80::f4da:bc9b:9cc1:2daf%26 的 Ping 統計資料:
        封包: 已傳送 = 4，已收到 = 4, 已遺失 = 0 (0% 遺失)，
    大約的來回時間 (毫秒):
        最小值 = 0ms，最大值 = 0ms，平均 = 0ms
    ```

9. 最後，我們可以透過`ssh`連接到 Raspberry Pi Zero W 2 上
    ```bash
    ssh pi.local
    ```
    ![connect](https://i.imgur.com/6pv5oL5.png)



## 結論
----------

我們從沒有外部螢幕和鍵盤的情況下，利用USB連線設定和訪問Raspberry Pi Zero W 2

但可惜的是，我這邊沒有測試成功Linux和MacOS，如果有人成功了，歡迎留言告訴我，我再補充上去🥲

Linux的坑滿多的，連上去還有可能會遇到找不到裝置的問題，所以我就放棄了😂


## 參考資料
----------
- [How to Connect a Raspberry Pi to a PC or Laptop Using USB](https://www.makeuseof.com/how-to-connect-raspberry-pi-to-laptop-or-pc-usb/)
- [Turning your Raspberry Pi Zero into a USB Gadget > Ethernet Gadget](https://learn.adafruit.com/turning-your-raspberry-pi-zero-into-a-usb-gadget/ethernet-gadget)
- [Windows/Install RNDIS driver](https://wiki.sipeed.com/hardware/en/maixsense/maixsense-a075v/install_drivers.html)
- [Raspberry Pi Zero W Headless setup – Windows 10 RNDIS Driver issue resolved](https://www.factoryforward.com/pi-zero-w-headless-setup-windows10-rndis-driver-issue-resolved/)