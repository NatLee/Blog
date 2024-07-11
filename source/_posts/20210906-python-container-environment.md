---
title: 使用Container來快速建立開發環境(以python為例)
categories: dev
tags:
  - docker
  - python
  - container
abbrlink: 440569875
date: 2021-09-06 00:00:00
---

![whale](https://www.docker.com/sites/default/files/d8/2019-07/horizontal-logo-monochromatic-white.png)

## 前言
----------

一次開發一次爽
一直開發一直慘
解決環境問題的一種好方法 ── Docker

<!--more-->

## 內容
----------

### 步驟

一開始先在 VScode Plugin 中找到並安裝 Remote Container 插件

![Remote container plugin](https://i.imgur.com/jGE3rXY.png)

然後，找一個你喜歡的地方，clone官方範例

`git clone https://github.com/Microsoft/vscode-remote-try-python`

 使用 VScode 打開它，VScode抓到設定檔後，右下角會跳出訊息問你要不要使用Container開啟，請按下**Reopen in Container**!

![Reopen in container](https://i.imgur.com/Ibkv1g5.png)

等待建立完成後，你會發現整個環境都在Cotainer內了，舒服！
接著注意左邊檔案可以發現一個資料夾**.devcontainer**
這是VScode建立容器參照的設定

![.devcontainer](https://i.imgur.com/ZxSGfeo.png)

點進去 **devcontainer.json** 可以看到有一行拿來指定port
這邊可以自行設定要publish到外部的port
換句話說就是你要使用的port
預設9000，但為了示範整個流程跟衝port，所以我這邊把它改成4000
其他還有一些執行相關的有趣設定可以研究 

![port](https://i.imgur.com/gOwm6zg.png)

接著由於 VScode 設定的關係
我們要去 **.vscode** 資料夾內查看他的設定
可以發現我們直接F5在VScode內偵錯時，他執行port不是我們剛剛指定的4000也因為這樣我們必須把20行改成4000
> 這邊有一些原因，是因爲上一步驟指定4000後，如果VScode沒有在這邊設定指定4000並且跑過一次，他會暫時鎖住4000 port 會造成 connection refuse，算是一個坑，但如果你沒有port相沖的問題，基本上開發用預設的9000測試就可以了

![setting](https://i.imgur.com/rsvco2q.png)

接著，儲存後按下 F5 執行 **app.py**! 

![demo page](https://i.imgur.com/X6r1kma.png)

### 補充說明

有時候總是會有那麼點原因導致你可能不想用vscode設定
而是想要直接在Container用單純的command去執行

那麼只要map好輸出的port也可以在底下的terminal直接run

`python app.py`

![](https://i.imgur.com/J3SGDOj.png)

![](https://i.imgur.com/c9XBSDH.png)

或是直接用 docker 進入container內 run 

![](https://i.imgur.com/MxmZLIb.png)

這邊就要說兩種做法了，你可以直接 **docker exec -it** 進去（這樣做的話，你得知道他幫你建立的容器叫什麼，可能要先 **docker ps -a** 一下） 也可以透過現在新版 docker 提供的 dashboard 進去

![](https://i.imgur.com/B0RyI5Z.png)
![](https://i.imgur.com/KyeUmAg.png)  


進去之後，再 cd 到 **/workspaces/vscode-remote-try-python** ，就可以看到你的程式了，其中 **vscode-remote-try-python** 是你的專案名稱

如果你程式有更動的話，像是我新增了一個套件
你可以選擇直接在容器內使用pip 安裝，也可以rebuild Container(建議) 來更新環境（當然，新的套件要註明在requirements.txt內）

簡單來說就是左下角點下去，上面會出現一個bar，按下第二個重新建立container （他會幫你蓋掉舊的Container，舒服！)

![](https://i.imgur.com/I7OyA5a.png)
![](https://i.imgur.com/Oy6au5Z.png)

如果我們直接關掉VScode 他也會幫你把container關閉

![](https://i.imgur.com/iPFmnXf.png)

下次再打開專案資料夾時，他就會問你要不要從container開啟，如果開啟的話，他就會幫你把container打開

### 結論

透過container開發真的很方便，麻煩的環境不用每次都顧慮
不過這樣做，整體步驟還是有點冗長，希望之後能夠發現更快部署的方法

## Reference
----------

[這次用到的截圖imgur存放區](https://imgur.com/a/GnQEPpi)

