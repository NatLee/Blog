---
title: 多版本CUDA及cuDNN管理
categories: develop
tags:
  - cuda
  - cudnn
  - deep_learning
  - ai
  - linux
  - ubuntu
abbrlink: 823759430
date: 2022-02-10 00:00:00
---

## 前言

---

最近復現別人 deep learning 相關的專案

常常會遇到不同專案有不同 CUDA 版本的問題

還有 package 相依問題也很麻煩

所以非常需要一個有效管理多 CUDA 版本的方法

<!-- more -->

## 內容

---

不斷更換 CUDA 版本真的很讓人煩躁

而且常常遇到每個人的專案用不同的版本

我就嘗試 google 搜尋了一下有沒有苦過的人

果然不出所料，一堆苦人

我從[Reference](#Reference)找到一個外國大佬的做法讓切換版本變得十分簡單

所以我打算直接引用他的做法當做一個記錄，幫助下一個苦（CU）人

### Install CUDA

安裝 CUDA 比較快，直接從[NVIDIA 官網](https://developer.nvidia.com/cuda-toolkit-archive)下載`runfile`的版本，並照得它上面的指令輸入安裝

這邊有兩點很重要要注意：

> 1. 不用從 CUDA-toolkit 裏面安裝驅動程式，請手動額外去安裝最新的驅動
> 2. 安裝時，請不要選擇建立`symbolic link`

底下是安裝的範例以 10.1 爲例

```bash
wget https://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run
sudo sh cuda_10.1.243_418.87.00_linux.run
```

```bash
 CUDA Installer
 - [ ] Driver
      [ ] 418.87.00
 + [X] CUDA Toolkit 10.1
   [X] CUDA Samples 10.1
   [X] CUDA Demo Suite 10.1
   [X] CUDA Documentation 10.1
   Options
   Install
```

進去`Options`

```bash
CUDA Toolkit
  Change Toolkit Install Path
  [ ] Create symbolic link from /usr/local/cuda
- [ ] Create desktop menu shortcuts
     [ ] All users
     [ ] Yes
     [ ] No
  [X] Install manpage documents to /usr/share/man
  Done
```

最後安裝完成就會吐出 Summary

```bash
===========
= Summary =
===========

Driver:   Not Selected
Toolkit:  Installed in /usr/local/cuda-10.1/
Samples:  Installed in /home/natlee/, but missing recommended libraries

Please make sure that
 -   PATH includes /usr/local/cuda-10.1/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-10.1/lib64, or, add /usr/local/cuda-10.1/lib64 to /etc/ld.so.conf and run ldconfig as root

To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-10.1/bin

Please see CUDA_Installation_Guide_Linux.pdf in /usr/local/cuda-10.1/doc/pdf for detailed information on setting up CUDA.
***WARNING: Incomplete installation! This installation did not install the CUDA Driver. A driver of version at least 418.00 is required for CUDA 10.1 functionality to work.
To install the driver using this installer, run the following command, replacing <CudaInstaller> with the name of this run file:
    sudo <CudaInstaller>.run --silent --driver

Logfile is /var/log/cuda-installer.log
```

### Install cuDNN

1. 從 NVIDIA 官網下載對應 CUDA 版本的 cuDNN，然後用`tar -xvf <CUDNN_ZIP_NAME>`解壓縮

   內容會長得像下面這樣

   ```bash
   $ tree -L 2 ./cuda
   .
   ├── cuda
   │   ├── include
   │   ├── lib64
   │   └── NVIDIA_SLA_cuDNN_Support.txt
   └── cudnn_install.sh

   3 directories, 2 files
   ```

2. 在解壓縮的 cuDNN 同目錄下，使用`touch cudnn_install.sh`新增腳本，並且把下面的指令貼進去

   ```bash
   read -p "CUDA-version: " ver

   cp cuda/include/cudnn.h /usr/local/cuda-"${ver}"/include

   cp cuda/lib64/libcudnn* /usr/local/cuda-"${ver}"/lib64

   chmod a+r /usr/local/cuda*/include/cudnn.h /usr/local/cuda*/lib64/libcudnn*

   echo include

   tree -L 1 /usr/local/cuda-"${ver}"/include | grep cudnn

   echo lib64

   tree -L 1 /usr/local/cuda-"${ver}"/lib64 | grep cudnn

   ```

   這個腳本是自動幫你把 cuDNN 丟到 CUDA 目錄下

   我真的不知道爲什麼 cuDNN 跟 CUDA 要分開下載，有夠麻煩

3. 更改權限後，run 安裝 cuDNN 的腳本

   ```bash
   chmod +x cudnn_install.sh & sudo ./cudnn_install.sh
   ```

4. 安裝完成後，我們可以新增一個切換 CUDA 版本的 function 在`~/.bashrc`或`~/.zshrc`裏面，以便下次直接 call 它來切換版本

   ```bash
   # add below to your env bash file.

   function _switch_cuda {
      v=$1
      export PATH=$PATH:/usr/local/cuda-$v/bin
      export CUDADIR=/usr/local/cuda-$v
      export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-$v/lib64
      nvcc --version
   }
   ```

   改完記得`source`一下

   它只要輸入版本號就能使用，裏面會自動幫你切換目前使用 CUDA 的路徑

   例如： `_switch_cuda 10.1`

5. 最後，可以確認看看`/usr/local`中的 CUDA 版本

   ```bash
   $ tree -L 1 /usr/local
   .
   ├── bin
   ├── cuda-10.0
   ├── cuda-10.1
   ├── cuda-10.2
   ├── cuda-9.0
   ├── cuda-9.1
   ├── cuda-9.2
   ├── etc
   ├── games
   ├── include
   ├── lib
   ├── man -> share/man
   ├── sbin
   ├── share
   ├── src
   └── texlive

   16 directories, 0 files
   ```

## 結語

---

自從用這個方法裝了多版本 CUDA 再搭配 mini-conda

整個開發環境都變舒服了

## Reference

---

[Installing multiple versions of cuda cudnn](https://notesbyair.github.io/blog/cs/2020-05-26-installing-multiple-versions-of-cuda-cudnn/)

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E5%A4%9A%E7%89%88%E6%9C%ACcuda%E5%8F%8Acudnn%E7%AE%A1%E7%90%86-e8f0f3c27583)
