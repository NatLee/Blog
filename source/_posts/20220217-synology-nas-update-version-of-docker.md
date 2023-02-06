---
title: Synology NAS Docker更新與遠端控制
categories: dev
tags:
  - synology
  - nas
  - docker
  - linux
abbrlink: b87de6f1
date: 2022-02-17 10:44:22
---

![docker_logo](https://d1.awsstatic.com/acs/characters/Logos/Docker-Logo_Horizontel_279x131.b8a5c41e56b77706656d61080f6a0217a3ba356d.png)

## 前言

---

NAS 大多被用來儲存或是分享檔案

但其實還可以有很多其他有趣的用途，像是當個 docker 容器伺服器

<!--more-->

## 內容

---

> _WAIT！請注意！直接更改 NAS 內的檔案都會有風險！_ > _請一定要知道你的每一步操作在做什麼！_

### 更新 Docker

首先，我們要確保 NAS 內的 docker 版本比較新的

因爲 Synology 官方都很念舊，docker 版本一直都不想更新

甚至現在都 2022 年了，docker 還停在 18 版

所以我們必須更新它

當然這個問題衆所皆知，國外也有網友自幹了一波更新腳本

可以參考一下這邊[Synology docker](https://github.com/markdumay/synology-docker)

我們依照它的教學，先以 ROOT 登入 NAS 後，把腳本 clone 下來

```bash
$ git clone https://github.com/markdumay/synology-docker.git
$ cd synology-docker
```

> 如果你沒有 GIT 指令，請參考之前的文章去安裝套件管理器

接着，我們就可以使用腳本更新 docker 了

> 請注意！如果你舊有 docker 有重要資料，請不要強制更新！

```bash
$ sudo ./syno_docker_update.sh update
```

更新成功後，我們可以確認一下 docker 版本

```bash
$ docker --version
Docker version 20.10.12, build e91ed57
```

到這邊就成功更新 docker 版本了！

### Docker 遠端設定

安裝好 docker 後，我們必須 expose docker 的遠端控制 port

所以，要編輯這個文件

```bash
$ vi /var/packages/Docker/etc/dockerd.json
```

把裏面改成下面這樣，主要是加上`hosts`

```json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2375"],
  "data-root": "/var/packages/Docker/target/docker",
  "log-driver": "json-file",
  "registry-mirrors": [],
  "group": "administrators"
}
```

接下來，把 NAS 的 docker 服務重啓

```
$ /var/packages/Docker/scripts/start-stop-status stop
$ /var/packages/Docker/scripts/start-stop-status start
```

重啓完成後，用瀏覽器訪問 NAS 的 port 2375，可以看到有回應（雖然 404）

```json
{ "message": "page not found" }
```

這樣代表我們已經成功 expose 了

之後，只要再用別台裝置的 docker 就能夠遠端在 NAS 上 run container 了

甚至是我們也可以用 portainer 去控制 NAS 的 docker

## 結語

---

其實這篇跟上篇套件管理有相關

爲了讓閒置的 NAS 有其他用途，只好想方設法在上面裝一些東西玩玩 😊

## Reference

---

[Synology docker](https://github.com/markdumay/synology-docker)
[Docker Host](https://blog.asper.one/2018/01/docker-docker-host-synology-ubuntu.html)

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/synology-nas-docker%E6%9B%B4%E6%96%B0%E8%88%87%E9%81%A0%E7%AB%AF%E6%8E%A7%E5%88%B6-2b43810ac8fe)
