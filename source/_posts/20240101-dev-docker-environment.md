---
title: 使用Docker快速建立GUI圖形化的深度學習開發環境
categories: dev
tags:
  - docker
  - console
  - container
abbrlink: c7016718
date: 2024-01-01 00:00:00
---

![GUI](https://raw.githubusercontent.com/NatLee/dev-dock/main/doc/desktop.png)

## 前言

從以前到現在，開發環境一直都是令人難以捉摸的問題，尤其是在不同的開發環境之間切換，更是令人頭痛

在這篇文章中，我們會 Docker 來建立一個圖形化的開發環境，並且可以在不同的電腦上快速的部署

---

<!--more-->

## 內容

---

> 只有在 x86/64 的 Linux 系統測試過

> 這篇文章完成的專案在[Dev Dock](https://github.com/NatLee/dev-dock)

讓我們把時間回到2018年，那時候 docker 還沒有那麼普及

我們在開發的時候，常常會遇到開發環境的問題

尤其是在不同的裝置上，因爲硬體的差異，更會遇到版本不同、套件無法編譯的狀況

我們就會採用虛擬機的方式，來建立一個開發環境

但是虛擬機的缺點就是，佔用的資源太多，而且啓動速度也很慢

尤其是硬體資源不足的狀況下，我們常常需要開關虛擬機

而且虛擬機在啓動的時候要設定一堆參數，還要設定網路跟連線方式，真的是很麻煩

那時候的菜鳥如我還在某間很窮的NAS公司上班

因爲硬體資源匱乏，開發又需要多環境，所以我就開始 survey 有什麼解決方案

於是乎，我就找到了 docker

![docker-logo](https://www.docker.com/wp-content/uploads/2023/08/logo-guide-logos-1.svg)

使用 docker 建立的容器，可以快速的啓動，而且佔用的資源也很少

基本上，我們可以把 docker 當成一個輕量級的虛擬機（這邊只是假設，實際上每個容器都只是一支程式）

我們只要在 `Dockerfile` 中，寫好我們要的環境並且 build 成獨立的 image 就可以快速部署

這邊放上已開源的專案（與當初弄的時候比，現在可能還是不太完整，但應該算是有進化過的版本XD）

- [Dev Dock](https://github.com/NatLee/dev-dock)

接下來，我們來拆解一下這個專案XD

這個專案主要是讓大家能夠快速使用，所以主目錄底下直接可以看到 `Dockerfile` 跟 `docker-compose.yml`兩個檔案

### Dockerfile 解說

> [Dockerfile](https://github.com/NatLee/dev-dock/blob/main/Dockerfile)

Dockerfile 是用來建立 image 的，我們可以在這邊寫入我們要的環境

這個檔案裏面指定了一些預設值，例如PORT位置、VNC的密碼（密碼竟然明碼公開，好孩子不要學XD）

以及安裝了一些開發需要用到的套件，像是圖形化界面Xfce、VNC、VS Code以及Mini Conda等等

有興趣可以看看[script的目錄](https://github.com/NatLee/dev-dock/tree/main/scripts)

裏面最重要的部分是安裝圖形化界面的[install-vnc-core.sh](https://github.com/NatLee/dev-dock/blob/main/scripts/3.install-vnc-core.sh)

這個腳本裏面可以切成三個部分，分別是安裝Xfce、TigerVNC以及NoVNC

```bash
#!/bin/bash
set -euxo pipefail

# 安裝 Xfce 的 UI
echo "Installing Xfce4 UI components"
apt-get -qq install -y supervisor xfce4 xfce4-terminal
apt-get -qq purge -y pm-utils xscreensaver*

# 安裝 VNC 功能（使用 TigerVNC ）
echo "Installing TigerVNC server..."
tar xzf $VNC_TOOL_DIR/tigervnc-1.10.1.x86_64.tar.gz --strip 1 -C /


# 安裝網頁版 VNC（使用 NoVNC ）
git clone --depth 1 https://github.com/novnc/noVNC.git $NO_VNC_DIR
chmod +x -v $NO_VNC_DIR/utils/*
ln -s $NO_VNC_DIR/vnc.html $NO_VNC_DIR/index.html
```

再來，每當我們啓動容器會執行[docker-entrypoint.sh](https://github.com/NatLee/dev-dock/blob/main/scripts/docker-entrypoint.sh)

首先，要根據環境的設定重新設定一下使用者的權限（可見[1.user-settings.sh](https://github.com/NatLee/dev-dock/blob/main/scripts/1.user-settings.sh)）

```bash
# Change user settings
$SCRIPTS_DIR/1.user-settings.sh
```

我們會再根據設定中，VNC的密碼來設定VNC的密碼

```bash
echo "$VNC_PW" | vncpasswd -f >> $PASSWD_PATH
chmod 600 $PASSWD_PATH
```

最後，依序啓動VNC Server、NoVNC以及 GUI 界面的 Xfce

#### VNC Server

```bash
# Start vncserver
vncserver -kill $DISPLAY || rm -rfv /tmp/.X*-lock /tmp/.X11-unix || echo "Remove old vnc locks to be a reattachable container"
vncserver $DISPLAY -depth $VNC_COL_DEPTH -geometry $VNC_RESOLUTION
```

#### NoVNC Service

```bash
# Start noVNC webclient
$NO_VNC_DIR/utils/novnc_proxy --vnc localhost:$VNC_PORT --listen $NO_VNC_PORT
```

#### GUI 界面

> [wm-startup.sh](https://github.com/NatLee/dev-dock/blob/main/xfce/wm-startup.sh)

```bash
# 關閉螢幕保護程式以及節能功能
xset -dpms &
xset s noblank &
xset s off &

# 啓動 Xfce（使用replace參數，可以讓我們啓動GUI，而不是開啓一個新的GUI）
# 然後把log輸出到 $HOME/wm.log
/usr/bin/startxfce4 --replace > $HOME/wm.log &
```

#### 使用 Dockerfile 來啓動

依照 Dockerfile 內容，我們可以使用以下指令來啓動容器

其中，我們可以使用 `-e` 來設定環境變數，例如 `VNC_PW` 來設定 VNC 的密碼

環境變數的名稱可以參考 Dockerfile 中的設定

> 注意，這邊指令中沒有 bind Device，所以沒有辦法控制主機的 GPU

> 如果要使用 GPU 的話，可以參考[docker-compose的部分](#docker-compose-解說)

```bash
docker run -d -p 12345:5901 -p 13579:6901 -p 24680:22 \
  -e VNC_PW=vnc-1234 \
  -e VNC_RESOLUTION=1600x900 \
  -e DEFAULT_USER='test' -e DEFAULT_USER_PASSWORD='test' \
  -e ROOT_PASSWORD='root' \
  --name test test-vnc-gui
```

啓動之後， Docker 服務中會出現一個名稱為 `test` 的容器（以 test-vnc-gui 爲基底建立的容器）

我們可以使用使用瀏覽器連線到 `http://localhost:13579` 來使用 noVNC 的網頁版圖形化界面

### Docker Compose 解說

> [docker-compose.yml](https://github.com/NatLee/dev-dock/blob/main/docker-compose.yml)

> [docker-compose.nvidia.yml](https://github.com/NatLee/dev-dock/blob/main/docker-compose.nvidia.yml)

除了直接 `docker run` 之外，我們還可以使用 `docker-compose up` 來啓動容器

`docker-compose.yml` 是一個設定檔，我們可以更方便的在裏面設定容器的參數，這邊以 `docker-compose.nvidia.yml` 爲例

因爲範例中有用到 NVIDIA 的 driver，所以我們要額外安裝[NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#installing-with-apt)

> 主機必須要有 NVIDIA 的 GPU 才能使用，如果沒有的話，直接使用 `docker-compose.yml` 即可

#### 安裝 NVIDIA Container Toolkit（Optional）

首先，我們要先把 NVIDIA 的 key 加入到 keyring 中

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

接著，更新 apt 並且直接安裝即可

```bash
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
```

安裝完成之後，我們就可以使用 docker-compose 來啓動容器

```yml
version: "3.9"

services:
  vnc:
    build: # 使用指定的 Dockerfile 來建立 image
      context: .
      dockerfile: ./Dockerfile
    container_name: test # 建立的容器的名稱
    ports:
      - "12345:5901" # VNC 的 port 對應
      - "13579:6901" # noVNC 的 port 對應
      - "24680:22" # SSH 的 port 對應
    environment:
      VNC_PW: test # VNC 的密碼
      VNC_RESOLUTION: 1600x900 # VNC 的解析度
      DEFAULT_USER: test # 預設使用者
      DEFAULT_USER_PASSWORD: test # 預設使用者的密碼
      ROOT_PASSWORD: root # root 的密碼
    privileged: true # 不一定要加，但是如果要使用GPU的話，就需要加上
    # =====================
    # GPU support
    # =====================
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

#### 使用 docker-compose 來啓動

- 使用 NVIDIA GPU

  ```bash
  docker-compose -f docker-compose.nvidia.yml build && docker-compose -f docker-compose.nvidia.yml up
  ```

- 不使用 NVIDIA GPU

  ```bash
  docker-compose build && docker-compose up
  ```

## 結論

雖然建立這種類虛擬機的違背 Docker 當初部署的本意

我們甚至把圖形化界面放到了容器中，所以我覺得這個做法算是滿大膽的

但是這樣的好處是我們可以透過網頁來使用圖形化界面

當然CLI的部分也還是一樣可以用SSH連線進去

順帶一提，這篇文章的 cover 就是實際跑起來的畫面

有了這個方法，我們就可以在不同的裝置上，快速的部署開發環境並且測試

若是同一台伺服器有這樣的 image ，也可以支援多人同時使用

所以這樣的嘗試還是有一定的價值的，希望可以幫助到更多開發者

不過，雖然 image build完後，docker run 很快

但這樣的環境還要慢慢使用 command line 去建立，還是不太方便

這邊預告下一集，

**『以Django爲後端開發集成式管理的Docker開發環境』**


---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E4%BD%BF%E7%94%A8docker%E5%BF%AB%E9%80%9F%E5%BB%BA%E7%AB%8Bgui%E5%9C%96%E5%BD%A2%E5%8C%96%E7%9A%84%E6%B7%B1%E5%BA%A6%E5%AD%B8%E7%BF%92%E9%96%8B%E7%99%BC%E7%92%B0%E5%A2%83-a2e8d5cfc804)






