
---
title: Synology NAS Docker更新與遠端控制
categories: dev
tags:
  - synology
  - nas
  - docker
  - linux
date: 2022-02-17 10:44:22
---

## 前言
----------

NAS大多被用來儲存或是分享檔案

但其實還可以有很多其他有趣的用途，像是當個docker容器伺服器

<!--more-->

## 內容
----------

> _WAIT！請注意！直接更改NAS內的檔案都會有風險！_
> _請一定要知道你的每一步操作在做什麼！_


### 更新Docker

首先，我們要確保NAS內的docker版本比較新的

因爲Synology官方都很念舊，docker版本一直都不想更新

甚至現在都2022年了，docker還停在18版

所以我們必須更新它

當然這個問題衆所皆知，國外也有網友自幹了一波更新腳本

可以參考一下這邊[Synology docker](https://github.com/markdumay/synology-docker)

我們依照它的教學，先以ROOT登入NAS後，把腳本clone下來

```bash
$ git clone https://github.com/markdumay/synology-docker.git
$ cd synology-docker
```

> 如果你沒有GIT指令，請參考之前的文章去安裝套件管理器

接着，我們就可以使用腳本更新docker了

> 請注意！如果你舊有docker有重要資料，請不要強制更新！

```bash
$ sudo ./syno_docker_update.sh update
```

更新成功後，我們可以確認一下docker版本

```bash
$ docker --version
Docker version 20.10.12, build e91ed57
```

到這邊就成功更新docker版本了！

### Docker遠端設定

安裝好docker後，我們必須expose docker的遠端控制port

所以，要編輯這個文件

```bash
vi /var/packages/Docker/etc/dockerd.json
```

把裏面改成下面這樣，主要是加上`hosts`

```json
{
+    "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2375"],
     "data-root" : "/var/packages/Docker/target/docker",
     "log-driver" : "json-file",
     "registry-mirrors" : [],
     "group": "administrators"
}
```

接下來，把NAS的docker服務重啓

```
$ /var/packages/Docker/scripts/start-stop-status stop
$ /var/packages/Docker/scripts/start-stop-status start
```

重啓完成後，用瀏覽器訪問NAS的port 2375，可以看到有回應（雖然404）

```json
{"message":"page not found"}
```

這樣代表我們已經成功expose了

之後，只要再用別台裝置的docker就能夠遠端在NAS上run container了

甚至是我們也可以用portainer去控制NAS的docker


## Reference
----------

[Synology docker](https://github.com/markdumay/synology-docker)
[Docker Host](https://blog.asper.one/2018/01/docker-docker-host-synology-ubuntu.html)