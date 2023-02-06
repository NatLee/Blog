---
title: 在Ubuntu上更新NVIDIA GPU驅動程式
categories: dev
tags:
  - nvidia
  - cuda
  - driver
  - ubuntu
  - linux
abbrlink: 6ff826ea
date: 2022-03-11 12:28:45
---

![](https://www.nvidia.com/content/dam/en-zz/Solutions/geforce/drivers/cut-graphics/nvidia-geforce-drivers-meta-image-1200x627.jpg)

## 前言

---

最近搞一波模型升級，結果部署的機器顯卡驅動程式太舊

導致沒辦法成功載入模型，只好升級一下

但根據經驗，之前踩過太多坑，記錄一下避免後來又掉進來

<!--more-->

## 內容

---

我們都知道有個指令叫`nvidia-smi`，可以看到顯卡驅動的版本

> NVIDIA-SMI 465.19.01
> Driver Version: 465.19.01
> CUDA Version: 11.3

> 注意上面的 CUDA 版本並非你實際使用的版本

我們另外可以透過`sudo lshw -C display`去看到顯示卡更詳細的資訊

```bash
  *-display
       description: VGA compatible controller
       product: NVIDIA Corporation
       vendor: NVIDIA Corporation
       physical id: 0
       bus info: pci@0000:01:00.0
       version: a1
       width: 64 bits
       clock: 33MHz
       capabilities: pm msi pciexpress vga_controller bus_master cap_list rom
       configuration: driver=nvidia latency=0
       resources: irq:16 memory:a3000000-a3ffffff memory:90000000-9fffffff memory:a0000000-a1ffffff ioport:3000(size=128) memory:c0000-dffff
```

或是我們也可以使用[這個指令](https://askubuntu.com/questions/72766/how-do-i-find-out-the-model-of-my-graphics-card)去看到顯卡是哪張

```bash
lspci | grep VGA
```

> 01:00.0 VGA compatible controller: NVIDIA Corporation GP104 [GeForce GTX 1070 Ti] (rev a1)

### 安裝步驟

接下來，進入主題

我們有兩種方式可以安裝 NVIDIA 的驅動程式

一種是直接使用`apt`指令，另一種是從官網下載驅動程式

1. 方法一：直接使用`apt`指令安裝

   我們要安裝 / 更新顯卡的驅動程式可以依照以下步驟去操作

   首先，起手式

   ```bash
   sudo apt update
   sudo apt upgrade
   ```

   > 注意，Upgrade 的時候，可能會提示你重開機，照着它操作就行了

   接着，我們要查看目前有哪些`nvidia`的驅動能用

   ```bash
   sudo apt-cache search nvidia-driver
   ```

   以下列出找到的部分結果

   ```bash
   nvidia-384 - Transitional package for nvidia-driver-390
   nvidia-384-dev - Transitional package for nvidia-driver-390
   nvidia-driver-390 - NVIDIA driver metapackage
   nvidia-headless-390 - NVIDIA headless metapackage
   nvidia-headless-no-dkms-390 - NVIDIA headless metapackage - no DKMS
   ...
   nvidia-driver-495 - Transitional package for nvidia-driver-510
   nvidia-driver-510 - NVIDIA driver metapackage
   ```

   如果這邊遇到找不到的問題，可以嘗試添加其他顯卡驅動 package 的來源

   ```bash
   sudo add-apt-repository ppa:graphics-drivers/ppa
   sudo apt update
   ```

   假設我今天選定了`nvidia-430`這個版本，就可以直接使用下面的指令安裝

   ```bash
   sudo apt install nvidia-430
   ```

   > 如果有舊的驅動，系統會提示說安裝前會把舊版驅動移除
   > 若驅動版本比較新，請手動移除
   > 用指令 `sudo apt list --installed | grep nvidia` 可以查詢已安裝的驅動版本

   安裝完成後，重新開機輸入就能`nvidia-smi`就能看到新版驅動了

2. 方法二：從官網下載安裝包

   我們要從官網下載驅動程式，可以從[下載頁面](https://www.nvidia.com/Download/index.aspx)找到選項

   這邊可以依照剛剛一開始找到的顯卡資訊填入，之後按下 SEARCH 按鈕就可以下載驅動程式了

   ![下載選項](https://i.imgur.com/ecANLXb.png)

   > LINUX X64 (AMD64/EM64T) DISPLAY DRIVER
   > Version: 515.57
   > Release Date: 2022.6.28
   > Operating System: Linux 64-bit
   > Language: English (US)
   > File Size: 346.53 MB

   接着把驅動程式上傳到目標電腦後，使用以下指令去安裝驅動程式

   ```bash
   sudo bash NVIDIA-Linux-x86_64-515.57.run
   ```

   安裝完後，重新開機輸入就能`nvidia-smi`就能看到新版驅動了

   ![driver](https://i.imgur.com/3xcZt7S.png)

### 額外步驟

假設安裝最新的驅動，遇到了一些問題

可以使用下面這個指令去移除舊版驅動

```bash
sudo apt-get purge nvidia*
```

如果想要移除新增的額外來源，可以使用這個指令

```bash
sudo apt-add-repository --remove ppa:graphics-drivers/ppa
```

## 總結

上面說明了兩種安裝方式

我個人覺得第二種比較好裝，畢竟就下載，然後丟上去一次到位

第一種則是牽扯到系統套件的相依性，可能會有其他衍生問題

不過第一種方式能夠包含 debug 把整個流程安裝完的話，功力也會提高吧 XD

## Reference

---

- [Ubuntu Linux Install Nvidia Driver (Latest Proprietary Driver)](https://www.cyberciti.biz/faq/ubuntu-linux-install-nvidia-driver-latest-proprietary-driver/)

- [ubuntu 系统安装 nvidia 驱动](https://blog.51cto.com/u_12630471/3705833)

- [How to install the latest Nvidia drivers on Ubuntu 16.04 Xenial Xerus](https://linuxconfig.org/how-to-install-the-latest-nvidia-drivers-on-ubuntu-16-04-xenial-xerus)
