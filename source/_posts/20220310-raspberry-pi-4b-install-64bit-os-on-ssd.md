---
title: 在SSD上爲Raspberry Pi 4B安裝64-bit的作業系統
categories: dev
tags:
  - raspberry
  - pi
  - lunux
  - os
abbrlink: 21d73346
date: 2022-03-10 11:58:39
---

![](https://upload.wikimedia.org/wikipedia/commons/d/d1/Raspberry_Pi_OS_Logo.png)

## 前言

---

上個月，Raspberry Pi 終於出了[官方正式版的 64-bit OS](https://www.raspberrypi.com/news/raspberry-pi-os-64-bit/)

我就想說順便也把手邊的樹莓派通通更新成 64-bit 好了

但夜路走多總是會踩到坑，於是就有了這篇

<!--more-->

## 內容

---

### 事前準備

首先，你需要的東西有這些：

- Raspberry Pi 4B 本體一台
- 能用的 SD 卡一張
- 能用的 SSD 一個
- 能讀 SD 卡的讀卡機一個
- SSD 轉接頭或裝置一個

> 這邊要注意 SD 卡的容量不要大於 SSD（應該也很少能夠大於 SSD）

### 開始行動

#### 燒錄 OS 到 SD 卡中

1. 我們先從官網下載這個燒錄用的軟體 [## Raspberry Pi Imager](https://www.raspberrypi.com/software/)，並且安裝

2. 打開這個軟體，選擇`Raspberry Pi OS（other)`

   ![](https://i.imgur.com/PtfmPdD.png)

3. 選擇`64-bit`的 OS

   ![](https://i.imgur.com/VVOsoil.png)

4. 請先確認你插入的 SD 卡裏面沒有需要的資料，再選擇它，並按下燒錄，然後等一陣子

#### 複製到 SSD 中

1. 我們的 SD 卡現在有 OS 了，直接把 SD 卡插到樹莓派上面，然後進行開機設定一下

   等最久的部分應該會是等它更新（建議這時候可以去把 VNC、SSH 功能打開）

   ![](https://i.imgur.com/VGi3dcC.png)

2. 更新完成後，我們重新啓動樹莓派

   ![](https://i.imgur.com/CT0AYz9.png)

3. 重啓完成後，我們打開終端並輸入下面指令更新

   ```bash
   sudo apt update
   sudo apt full-upgrade
   ```

   因爲一開始開機導引有設定過，所以這邊不會跳太多東西

4. 接着，我們使用下面指令更改一下`eeprom`更新的狀態

   ```bash
   sudo nano /etc/default/rpi-eeprom-update
   ```

   改成這樣

   ```yaml
   FIRMWARE_RELEASE_STATUS="stable"
   ```

5. 使用下面指令更新`eeprom`

   ```bash
   sudo rpi-eeprom-update -a
   ```

   如果沒什麼問題的話，這邊的輸出大概會長這樣

   ```bash
   pi@raspberrypi:~ $ sudo rpi-eeprom-update -a
   *** INSTALLING EEPROM UPDATES ***

   BOOTLOADER: update available
      CURRENT: Thu 02 Dec 2021 11:08:03 AM UTC (1638443283)
   	LATEST: Tue 08 Feb 2022 05:24:46 PM UTC (1644341086)
      RELEASE: stable (/lib/firmware/raspberrypi/bootloader/stable)
   			Use raspi-config to change the release.

     VL805_FW: Dedicated VL805 EEPROM
   	 VL805: up to date
      CURRENT: 000138a1
   	LATEST: 000138a1
      CURRENT: Thu 02 Dec 2021 11:08:03 AM UTC (1638443283)
   	UPDATE: Tue 08 Feb 2022 05:24:46 PM UTC (1644341086)
   	BOOTFS: /boot

   EEPROM updates pending. Please reboot to apply the update.
   To cancel a pending update run "sudo rpi-eeprom-update -r".

   ```

   注意：如果沒有做這個步驟的話，後面 copy 到 SSD 開機會出現下面的錯誤

   > `mmc1:Controller never released inhibit bit(s)....`

6. 重新啓動

   ```bash
   sudo reboot
   ```

7. 我們終於可以插上 SSD（記得要插到`USB3.0`的孔），使用系統工具去 copy

   ![](https://i.imgur.com/KpHAAmt.png)

   ![](https://i.imgur.com/2sckJrI.png)

   這邊要等待一段時間讓它複製（畢竟 SD 卡很慢）

   或是你可以使用指令去 copy sd 卡的內容到 SSD

   ```bash
   df -h  # View the name of the storage device
   #/dev/mmcblk0 by sd card   /dev/mmcblk0 by ssd
   sudo dd if=/dev/mmcblk0 of=/dev/sda bs=4M
   ```

8. 關機後，把 SD 卡拔掉再啓動就可以使用 SSD 正常進入到桌面了

   ![](https://i.imgur.com/8DSOvmU.png)

   到這邊就等於是成功啓動了，我們可以跟 SD 卡說掰掰了

   > 如果這一步開不起來的話，很有可能是供電的問題
   > 也就是 SSD 轉接吃的電量太大導致開不了機
   > 所以需要使用電源供應的孔來充電
   > 像是筆者測試[X.820 測試板](https://wiki.geekworm.com/X820)與[Argon 整合 SSD 的殼](https://www.youtube.com/watch?v=Tgrka088ZFk)測試
   > 會發現 Argon 的殼裝 SSD 較 X.820 來說，容易開不起來

9. (Optional) 我們可以安裝測速程式來試試看 SSD 的速度

   ```bash
   sudo apt install -y hdparm
   sudo hdparm -tT /dev/sda
   ```

   輸出會長這樣

   ```bash
   pi@raspberrypi:~ $ sudo hdparm -tT /dev/sda

   /dev/sda:
    Timing cached reads:   2178 MB in  2.00 seconds = 1091.06 MB/sec
    Timing buffered disk reads: 828 MB in  3.05 seconds = 271.12 MB/sec
   ```

   另外，有興趣可以測試原本 SD 卡的效能

   ```bash
   sudo hdparm -tT /dev/mmcblk0p1
   ```

   輸出會類似這樣

   ```bash
   /dev/mmcblk0p1:
    Timing cached reads:   1830 MB in  2.00 seconds = 915.81 MB/sec
    Timing buffered disk reads: 132 MB in  3.01 seconds =  43.88 MB/sec
   ```

   從上面可以看到讀取速度差滿多的

## 結語

---

這次我們從無到有安裝官方提供的 64-bit OS 到 SSD 上

其中有一些坑如`eeprom`的版本及`電供問題`

發生的時候都很難知道問題出在哪裏

像是電供的供電太少，導致電量不足沒辦法開機

讓我一直以爲是我的 OS 刷錯，瘋狂重刷沒改善

突然改用比較高瓦數的電供來供電才解決問題

總之一切都是坑，希望大家可以完美安裝 64-bit 的 OS

## 這次的圖片集中處

---

[都放在這邊 imgur](https://imgur.com/a/fcYqsxc)

## Reference

---

[Troubleshooting and solution to the failure of raspberry pie 4B boot system through external SSD hard disk](https://chowdera.com/2021/03/20210304183303424a.html)

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E5%9C%A8ssd%E4%B8%8A%E7%88%B2raspberry-pi-4b%E5%AE%89%E8%A3%9D64-bit%E7%9A%84%E4%BD%9C%E6%A5%AD%E7%B3%BB%E7%B5%B1-4cd7f93799f3)
