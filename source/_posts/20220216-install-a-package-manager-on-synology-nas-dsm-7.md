
---
title: 在群暉Synology NAS DSM7.0上安裝套件管理器
categories: dev
tags:
  - synology
  - nas
  - linux
date: 2022-02-16 22:15:54
---

![entware](https://repository-images.githubusercontent.com/123925061/fe5a0600-f91a-11ea-8914-601e3e722062)

## 前言
----------

我們都知道S社的DSM是一個難搞的東西，強制佔用又鎖port，還會擋東擋西

但物盡其用，我還是想在上面裝套件，至少不要連git都沒辦法用吧

<!--more-->

## 內容
----------

> _WAIT！請注意！直接更改NAS內的檔案都會有風險！_
> _請一定要知道你的每一步操作在做什麼！_

某次我SSH連線進NAS的時候發現Git沒辦法使用

而且官方Docker套件又弄得很廢

於是萌生出有沒有辦法在上面自己安裝套件的想法

然後我就使用了`apt`大法

結果發現 幹！原來OS不是debian系列

花時間上網搜了一波發現大多資料都是DSM6以前的，DSM7.0以上的資料超級少

最後找到[這篇文章](https://windix.medium.com/install-package-via-opkg-entware-for-synology-dsm-7-9b9994415b2d)說可以用`opkg`來安裝

我就查一下發現它裏面說的`Entware`是邊緣嵌入式系統用的套件管理器

於是跟着連結往深處尋找發現這個安裝流程[Entware install guide](https://github.com/Entware/Entware/wiki/Install-on-Synology-NAS)

因爲我NAS的CPU是intel x64，所以我直接執行了這個腳本

```bash
wget -O - https://bin.entware.net/x64-k3.2/installer/generic.sh | /bin/sh
```

它就會拉下來開始裝一些基本工具，成功會有下面訊息

```bash
...
Info: Congratulations!
Info: If there are no errors above then Entware was successfully initialized.
Info: Add /opt/bin & /opt/sbin to $PATH variable
Info: Add "/opt/etc/init.d/rc.unslung start" to startup script for Entware services to start
Info: Found a Bug? Please report at https://github.com/Entware/Entware/issues

```

接着用瀏覽器去個人NAS的首頁找到這個地方

> 設定 -> 服務 -> 任務排程器

按下新增`觸發任務`裏面的`使用者定義程式碼`

裏面照着下面填寫

```
一般設定

> 任務名稱：Entware
> 使用者帳號：root
> 事件：開機
> 先行任務：

[v]已啓動

```

接着按上面的`任務設定`的tab把下面程式碼貼上以確保每次重開機都能夠先載入套件管理的工具

```bash
#!/bin/sh

# Mount/Start Entware
mkdir -p /opt
mount -o bind "/volume1/@Entware/opt" /opt
/opt/etc/init.d/rc.unslung start

# Add Entware Profile in Global Profile
if grep  -qF  '/opt/etc/profile' /etc/profile; then
	echo "Confirmed: Entware Profile in Global Profile"
else
	echo "Adding: Entware Profile in Global Profile"
cat >> /etc/profile <<"EOF"

# Load Entware Profile
. /opt/etc/profile
EOF
fi

# Update Entware List
/opt/bin/opkg update
```

按下確定後就可以重開機了！


開機完成後，我們可以試試看安裝git

```bash
root@Nat-Nas:~# opkg install git-http
Installing git-http (2.33.1-1) to root...
Downloading http://bin.entware.net/x64-k3.2/git-http_2.33.1-1_x64-3.2.ipk
Installing git (2.33.1-1) to root...
Downloading http://bin.entware.net/x64-k3.2/git_2.33.1-1_x64-3.2.ipk
Installing ca-bundle (20211016-1) to root...
Downloading http://bin.entware.net/x64-k3.2/ca-bundle_20211016-1_all.ipk
Installing libcurl (7.80.0-1) to root...
Downloading http://bin.entware.net/x64-k3.2/libcurl_7.80.0-1_x64-3.2.ipk
Configuring ca-bundle.
Configuring libcurl.
Configuring git.
Configuring git-http.
```

終於搞定了！

接下來就可以用GIT去clone一些專案


## 結語
----------

其實在DSM6的時候，官方套件管理可以安裝Git功能

甚至是連Gitlab都能在官方套件內安裝

後來看起來是Synology不想更新這一塊了

所以乾脆直接把套件抽掉

好險非官方社群還有人在研究，不然閒置一台機器真的滿浪費的


## Reference
----------

[Entware install guide](https://github.com/Entware/Entware/wiki/Install-on-Synology-NAS)
