---
title: 解決SynologyNAS上Gitlab發生的502錯誤
categories: gitlab
tags:
  - synology
  - nas
  - gitlab
abbrlink: 3672378701
date: 2018-08-30 00:00:00
---

![](https://i.imgur.com/F87nc59.jpg)
<center>*OVERLORD*</center>

## 前言
----------

今天我家跳電了
於是NAS就重啟了

<!--more-->

## 似乎出了點問題
----------

回家後，我打開NAS檢查有沒有錯誤
發現它自帶的一致性檢查通過，不愧是鍍金的紅標硬碟，十分可靠

但我打開Gitlab的頁面確認時，出現一個致命的錯誤 :/

Oh!

![](https://i.imgur.com/eVL3B7d.jpg)

502，居然是502
怎麼會這樣

於是我重啟兩次Gitlab服務，依然502
看來效果並不超群

那麼就只好到docker裡面看看了

![](https://i.imgur.com/97XYNKI.png)

太慘了吧
而且還是unicorn

<iframe width="560" height="315" src="https://www.youtube.com/embed/YPtVSuy_wuU" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

你才unicorn，你全家都unicorn :(

後來只好向Google大神求救
於是發現原來是unicorn不斷地呼叫已死去的同胞

不斷跌倒，不斷站起
真是一個勵志的故事，而且造就不凡的CPU使用量

但我們還是得解決這個問題
只好進入Gitlab的docker container終端
毫不留情的砍掉犧牲者的PID名單

```bash
    root@synology_gitlab:/home/git/gitlab# rm /home/git/gitlab/tmp/pids/unicorn.pid
```

此時，重啟Gitlab服務，等個幾分鐘
天空又如同以往一樣清澈

## 後記
----------

我後來去看Gitlab的container日誌發現一句話

![](https://i.imgur.com/MrnVKll.png)

我想大概是停電造成關閉不完全以致殘留檔案造成後來復電重啟時開檔錯誤吧 :'(

## Reference
----------

[Synology GitLab error 502](https://blog.stead.id.au/2017/03/synology-gitlab-error-502.html)
