---
title: 多版本CUDA及cuDNN管理
date: 2022-02-10
categories: develop
tags: [cuda, cudnn, deep_learning, ai, linux, ubuntu]
---

![](https://upload.wikimedia.org/wikipedia/commons/5/59/CUDA.png)

## 前言
----------

最近復現別人deep learning相關的專案

常常會遇到不同專案有不同CUDA版本的問題

還有package相依問題也很麻煩

所以非常需要一個有效管理多CUDA版本的方法


<!-- more -->



## 內容
----------

不斷更換CUDA版本真的很讓人煩躁

而且常常遇到每個人的專案用不同的版本

我就嘗試google搜尋了一下有沒有苦過的人

果然不出所料，一堆苦人

我從[Reference](#Reference)找到一個外國大佬的做法讓切換版本變得十分簡單

所以我打算直接引用他的做法當做一個記錄，幫助下一個苦（CU）人


### Install CUDA


安裝CUDA比較快，直接從[NVIDIA官網](https://developer.nvidia.com/cuda-toolkit-archive)下載`runfile`的版本，並照得它上面的指令輸入安裝

這邊有兩點很重要要注意：

> 1. 不用從CUDA-toolkit裏面安裝驅動程式，請手動額外去安裝最新的驅動
> 2. 安裝時，請不要選擇建立`symbolic link`


### Install cuDNN


1. 從NVIDIA官網下載對應CUDA版本的cuDNN，然後用`tar -xvf <CUDNN_ZIP_NAME>`解壓縮
	
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

2. 在解壓縮的cuDNN同目錄下，使用`touch cudnn_install.sh`新增腳本，並且把下面的指令貼進去

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

	這個腳本是自動幫你把cuDNN丟到CUDA目錄下
	
	我真的不知道爲什麼cuDNN跟CUDA要分開下載，有夠麻煩


3. 更改權限後，run安裝cuDNN的腳本

	```bash
	chmod +x cudnn_install.sh & sudo ./cudnn_install.sh
	```

4.  安裝完成後，我們可以新增一個切換CUDA版本的function在`~/.bashrc`或`~/.zshrc`裏面，以便下次直接call它來切換版本

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

	它只要輸入版本號就能使用，裏面會自動幫你切換目前使用CUDA的路徑

	例如： `_switch_cuda 10.1`.

5. 最後，可以確認看看`/usr/local`中的CUDA版本

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
----------

自從用這個方法裝了多版本CUDA再搭配mini-conda

整個開發環境都變舒服了


## Reference
----------

[Installing multiple versions of cuda cudnn](https://notesbyair.github.io/blog/cs/2020-05-26-installing-multiple-versions-of-cuda-cudnn/)