
---
title: 在Ubuntu上更新NVIDIA GPU驅動程式
categories: dev
tags:
  - nvidia
  - cuda
  - driver
  - ubuntu
  - linux
date: 2022-03-11 12:28:45
---

## 前言
----------

最近搞一波模型升級，結果部署的機器顯卡驅動程式太舊

導致沒辦法成功載入模型，只好升級一下

但根據經驗，之前踩過太多坑，記錄一下避免後來又掉進來

<!--more-->

## 內容
----------

我們都知道有個指令叫`nvidia-smi`，可以看到顯卡驅動的版本

> NVIDIA-SMI 465.19.01
> Driver Version: 465.19.01
> CUDA Version: 11.3

> 注意上面的CUDA版本並非你實際使用的版本

我們另外可以透過`sudo lshw -C display`去看到顯示卡更詳細的資訊

```
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

### 安裝步驟

接下來，進入主題

我們要安裝 / 更新顯卡的驅動程式可以依照以下步驟去操作

首先，起手式

```
sudo apt update
sudo apt upgrade
```

> 注意，Upgrade的時候，可能會提示你重開機，照着它操作就行了

接着，我們要查看目前有哪些`nvidia`的驅動能用

```bash
sudo apt-cache search nvidia-driver
```

以下列出找到的部分結果

```
nvidia-384 - Transitional package for nvidia-driver-390
nvidia-384-dev - Transitional package for nvidia-driver-390
nvidia-driver-390 - NVIDIA driver metapackage
nvidia-headless-390 - NVIDIA headless metapackage
nvidia-headless-no-dkms-390 - NVIDIA headless metapackage - no DKMS
...
nvidia-driver-495 - Transitional package for nvidia-driver-510
nvidia-driver-510 - NVIDIA driver metapackage
```

如果這邊遇到找不到的問題，可以嘗試添加其他顯卡驅動package的來源

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
```

假設我今天選定了`nvidia-470`這個版本，就可以直接使用下面的指令安裝

```bash
sudo apt install nvidia-470
```

> 如果有舊的驅動，系統會提示說安裝前會把舊版驅動移除

安裝完成後，重新開機輸入就能`nvidia-smi`就能看到新版驅動了

### 額外步驟

有時候我們安裝最新的驅動，會遇到一些問題

可以使用下面這個指令去移除它

```bash
sudo apt-get purge nvidia*
```

如果想要移除剛剛有新增的額外來源，可以使用這個指令

```bash
sudo apt-add-repository --remove ppa:graphics-drivers/ppa
```


## Reference
----------

[Ubuntu Linux Install Nvidia Driver (Latest Proprietary Driver)](https://www.cyberciti.biz/faq/ubuntu-linux-install-nvidia-driver-latest-proprietary-driver/)
[ubuntu系统安装nvidia驱动](https://blog.51cto.com/u_12630471/3705833)