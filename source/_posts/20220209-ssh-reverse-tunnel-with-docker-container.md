---
title: 使用docker container建立SSH反向通道穿透內網連接內部裝置
categories: blog
tags:
  - docker
  - ssh
  - container
  - tunnel
abbrlink: 509187850
date: 2022-02-09 00:00:00
---

![](https://raw.githubusercontent.com/linuxserver/docker-templates/master/linuxserver.io/img/linuxserver_medium.png)

## 前言

---

之前一直很想寫反向通道(reverse tunnel)的流程，但過程很難描述

剛好最近在 docker 上看到好用的 openssh-server 的 image

結果一試就上手，於是就有這篇文章了

<!--more-->

## 步驟說明

---

> 首先，你需要一個`位於外網且可以透過SSH連線的裝置`
>
> 如果沒有，請按上一頁

我今天看到一個 image [linuxserver/openssh-server](https://hub.docker.com/r/linuxserver/openssh-server)

它把 openssh-server 的部分都做好了

所以我們可以直接用這個 image 去 run 一個 container

也就是寫一份 docker-compose 的文件就搞定了

接下來，我們會分別討論三個裝置上的設定

- 外網裝置 A
- 內網跳板裝置 B
- 透過跳板 B 連至內網裝置 C

步驟簡略來說會長成下面這個樣子：

- 內網的跳板裝置 B 會透過反向去連接在外網的 A（所以 B 在內網只要可以訪問外網就行了），接著 A 會使用另外一個 port 去 forward 裝置 B 的 SSH port

> A(SSH port) <--- B

這邊有點不太好懂，舉例來說就是假設 A 的 SSH port 是 1984，除了 1984 外，我們在路由器上設定開放 A 裝置的另一個 port 1994

那麼，B 反向到 A 的時候，我們可以利用 1994 這個 port 去讓別人連接 B 的 SSH port

- 我們透過某一可連外網的裝置（隨便一個都行）去連接 A 的反向專用 port 即可到達在內網的裝置 B

> 某裝置 ---> A(反向用 port) ---> B

- 因爲裝置 B 在內網，所以我們可以使用 B 來訪問所有在內網的裝置，例如 C

> 某裝置 ---> A(反向用 port) ---> B ---> C

### 外網裝置 A

首先，我們先在外網的機器 A 先佈置能能夠進行被反向連接的 server

我們基於 docker hub 上面官方的文件做修改

```yml
version: "3"
services:
  openssh-server:
    restart: unless-stopped
    image: lscr.io/linuxserver/openssh-server
    container_name: reverse-tunnel-outside-server
    hostname: nat-tunnel-outside-server #optional
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Taipei
      #- PUBLIC_KEY=yourpublickey #optional
      #- PUBLIC_KEY_FILE=/path/to/file #optional
      #- PUBLIC_KEY_DIR=/config/.ssh #optional
      #- PUBLIC_KEY_URL=https://github.com/username.keys #optional
      - SUDO_ACCESS=true #optional
      #- PASSWORD_ACCESS=false #optional
      #- USER_PASSWORD=password #optional
      #- USER_PASSWORD_FILE=/path/to/file #optional
      - USER_NAME=natlee #optional
    volumes:
      - ./ssh_setting:/config
      - ./root_ssh_setting:/root/.ssh
    ports:
      - 1984:2222
      - 1994:1994
```

主要比較有意義的改動是`volumes`跟`port`

> 有些版本的 docker 會出現無法 binding volumes 的問題，可以先自己用`mkdir`建立資料夾

因爲它的 image 包好的 SSH port 是`2222`，那麼我們把外部`1984`對應進去

另外，我們需要多開一個 port 讓內網裏面做反向的裝置能夠連到這臺 server

所以多開了一個 port `1994`

> 這邊記得在路由器幫這兩個 port 做 forwarding，建議內外相同 port 就行了

接着，我們就執行

```
docker-compose up
```

記得，這邊沒有使用`-d`去做背景執行是因爲等等 debug 比較方便

我們需要修改 openssh-server 的設定檔`sshd_config`

打開剛剛指定目錄中的`./ssh_setting/ssh_host_keys/sshd_config`去把這三個設定做更改

```
AllowTCPForwarding yes # 有些教學沒說要開這個，但不開它就不讓你forwarding了
GatewayPorts yes # 這個是讓你能夠轉port用的，一定要開

ChallengeResponseAuthentication no # 安全需求，只開放public key登入
```

重新執行

```
docker-compose up --force-recreate
```

完成這個步驟後

我們就可以放著不管，用背景執行這個 container 了

#### Extra Step

這邊是額外的連線測試，不做也不影響後面的流程

可以使用 local 做連線測試

```bash
❯ ssh natlee@localhost -p 1984
The authenticity of host '[localhost]:2000 ([::1]:1984)' can't be established.
ECDSA key fingerprint is SHA256:g73WW30kTFqdUvi/mCbIgrW4dnadPiW7ipopK9h6/zk.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[localhost]:1984' (ECDSA) to the list of known hosts.
natlee@localhost: Permission denied (publickey,keyboard-interactive).
```

於是，失敗了，因爲我們只能用 key 進行連線！

那麼要如何測試連線呢？

我們先找一對金鑰（如果沒有的話，可以用`ssh-keygen`生成）

再把剛找的公鑰（通常預設會在`~/.ssh/id_rsa.pub`）放到`./ssh_setting/.ssh/authorized_keys`內

內容類似長這樣的東西：

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDs13dOFtSQ8OJjIIZiVKWTIUbEvy+ZIcswOiXCb4q4nIK80yNyuDOVcbzQyyS0Q0z0UUTpf+TR5Rs4Ox0B8baffTC0CvFHQhwqpgvz/8d179fmUCyDb9yrdgdhSlN4zkMdw8A2P8SQ+3cc+gIQBvj/jo414kynz4zaoGseFxBOpVKc+dW1Y8m3G4wfBB0QpcyxYZ9vkAhrIYyPrIK9EP7LXTlJyQ7gMadJ4eUkMiBSnqfJYxPcYDIeK/uHYyGgpsJQUaxZ8JIqkD2kdmBPX8NHGA1O2VF2UyiJKuRVJEg/oW7YoCJHs81Gj+bl9HdnBC5CEMWckLPLRtfFkW3u9F6PNto9fs48L1dU8MGm6KAirX2GPydmwRS6yh9i32NT5J70izi28cX1IhJn2PtY75aZAPuU2NspsJ1cy4cbd35tJLB0H0= root@test
```

再用 SSH 連線一次！

```bash
❯ ssh natlee@localhost -p 1984
Welcome to OpenSSH Server

nat-tunnel-outside-server:~$
```

到這邊算是外網機器佈置成功了！

執行中的時候，資料夾內的內容物長這樣

```
.
├── docker-compose.yml
├── root_ssh_setting
└── ssh_setting
    ├── custom-cont-init.d
    ├── custom-services.d
    ├── logs
    │   ├── loginfo.txt
    │   └── openssh
    │       ├── current
    │       ├── lock
    │       └── state
    ├── sshd.pid
    └── ssh_host_keys
        ├── sshd_config
        ├── ssh_host_dsa_key
        ├── ssh_host_dsa_key.pub
        ├── ssh_host_ecdsa_key
        ├── ssh_host_ecdsa_key.pub
        ├── ssh_host_ed25519_key
        ├── ssh_host_ed25519_key.pub
        ├── ssh_host_rsa_key
        └── ssh_host_rsa_key.pub

7 directories, 15 files
```

### 內網跳板裝置 B

內網裝置需要的東西就比較多了

但還是老樣子，我們在內網裝置上起一個 openssh-server

基於 docker hub 上面官方的文件做修改

```yml
version: "3"
services:
  openssh-server:
    restart: unless-stopped
    image: lscr.io/linuxserver/openssh-server
    container_name: reverse-tunnel-inside-bridge
    hostname: nat-tunnel-inside-bridge #optional
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Taipei
      #- PUBLIC_KEY=yourpublickey #optional
      #- PUBLIC_KEY_FILE=/path/to/file #optional
      #- PUBLIC_KEY_DIR=/config/.ssh #optional
      #- PUBLIC_KEY_URL=https://github.com/username.keys #optional
      - SUDO_ACCESS=true #optional
      #- PASSWORD_ACCESS=false #optional
      #- USER_PASSWORD=password #optional
      #- USER_PASSWORD_FILE=/path/to/file #optional
      - USER_NAME=natlee #optional
    volumes:
      - ./ssh_setting:/config
      - ./root_ssh_setting:/root/.ssh
```

改完之後直接 run 一波

> 有些版本的 docker 會出現無法 binding volumes 的問題，可以先自己用`mkdir`建立資料夾

內網的裝置就不用給 ports，給了也只有內網電腦能連進去

```bash
docker-compose up
```

run 完會看到它的啓動畫面，然後就跟剛剛一樣生一堆文件出來

我們接下來要配置一些文件讓它能夠連到 server 且斷線後能夠重連

- 建立安裝 autossh 的腳本

  使用`sudo nano ./ssh_setting/custom-cont-init.d/install.sh`將以下內容貼上

  ```bash
  apk add --no-cache autossh
  ```

- 建立啓動 autossh 的腳本

  使用`sudo nano ./ssh_setting/custom-services.d/autossh.sh`將以下內容貼上

  ```bash
  #!/bin/bash

  echo "Start AutoSSH"

  /usr/bin/autossh \
      -M 6767 \
      -o "StrictHostKeyChecking=no" \
      -o "ServerAliveInterval 15" \
      -o "ServerAliveCountMax 3" \
      -p 1984 \
      -NR '*:1994:localhost:2222' \
      natlee@<YOUR_DOMAIN> \
      -i ~/.ssh/id_rsa
  ```

  > 這邊請把`<EXAMPLE_DOMAIN>`改成你自己連接外網裝置的 domain 或 IP
  > 其中，`StrictHostKeyChecking`用來自動新增`known_host`，避免要手動連接一次 A

接着，我們直接再 run 一次

```bash
docker-compose up
```

然後就噴錯了

```
reverse-tunnel-inside-bridge | Warning: Identity file /root/.ssh/id_rsa not accessible: No such file or directory.
reverse-tunnel-inside-bridge | Host key verification failed.
```

爲什麼它會說無法存取`/root/.ssh/id_rsa`？

因爲它必須要靠 key 來連到 server，但它並沒有 key pairs

所以我們得幫他產生一組

我們先別暫停 container，而是直接使用這個指令去生成 key pairs

```bash
docker exec -it reverse-tunnel-inside-bridge ssh-keygen
```

然後輸出會長成下面這樣

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:YYaMzm7rD4dAQlREdwzOMufrbe5KLwc1z554ePFIvsc root@nat-tunnel-inside-bridge
The key's randomart image is:
+---[RSA 3072]----+
|.o++ oo.         |
|.   +o.o         |
|. .o.+o +        |
| o o=  = .       |
|  . o.. S        |
|   o o.  =       |
|    =oo * *      |
|   .o=o= B E     |
|   .o+O=o.o      |
+----[SHA256]-----+
```

但這樣是不夠的，因爲我們外網的裝置只能使用 key 登入

這時候，我們就必須把公鑰檔案`~/.ssh/id_rsa.pub`的內容複製出來（直接進去用`cat`指令就行了）

```bash
docker exec -it reverse-tunnel-inside-bridge /bin/bash
root@nat-tunnel-inside-bridge:/# cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDdF4MGnC4XmCPR8QLy0gVJTQb9kf4jM5og0Q1ggO3y8wr8LMTdAufnatfj0mYq/QkCiVCQvfFT+e+f+1AKu0UCCokFehv6etUnyaHqQWDtrcXjT6yxw9N7njCrIv/mFHnttQ1z5Ri2ULOnoWva6hOabvUkabxDwwxz90HWKo94/xbtZsnU1kEmqyMJhJ2exSa+kSxUwKeyLX0HYCmiD5H1X+eqZMVbo3boDID3Pp2NFVFrGDP6NOwmBAVpd2rUHmL2AdKfcefvxmZKSyokp56J8pVg/MgzMEk7sEzZr2uP6klYNWpAsFaBJojrlQuUuGC1nibZmejRIEmuhcD+Wcssls8umB51L+0Hi+nZKU6lQXOZryW1xNXmS3h9DpeZTSNk4lmyJDuQ+r2JVLTNbyt1UvuGUotBuluDnNWwst8KWG8CAPjv+A/qMZ/dU3Rg9PEDQ8hXMccipq9lgq6bxrLXJ2gOKldTYXFhqhnyQs1b4AOXk7pwXaYo8vsWWkiJglU= root@nat-tunnel-inside-bridge
```

這串長長的公鑰就可以貼到我們外網 server A 的`./ssh_setting/.ssh/authorized_keys`中

這時候看回去 container 輸出就沒有再跳錯誤了！

再來是安全問題，我們可以再打開目錄中的`./ssh_setting/ssh_host_keys/sshd_config`去把這個設定做更改

```
ChallengeResponseAuthentication no # 安全需求，只開放public key登入
```

到這邊內網裝置 B 已經算是設定完成了

接下來，我們把自己第三方位置的 PC 公鑰複製到內網裝置 B 的`ssh_setting/.ssh/authorized_keys`後

我們從 PC 端使用 SSH 連接一開始有提到外網裝置多開的另外一個 port `1994`去做測試

```bash
❯ ssh natlee@<EXAMPLE_DOMAIN> -p 1994
Welcome to OpenSSH Server

nat-tunnel-inside-bridge:~$
```

成功連到裝置 B，也就是完美穿透防火牆，直接成功打洞進到內網了！

### 透過跳板 B 連至內網裝置 C

這邊的裝置 C，我使用樹莓派當範例

我們可以利用`~/.ssh/config`去設定 SSH 的快速連線方式，如下面的例子

```
Host reverse-tunnel-outside-server
    HostName <EXAMPLE_DOMAIN>
    User natlee
    port 1984
    Compression yes

Host reverse-tunnel-inside-bridge
    HostName <EXAMPLE_DOMAIN>
    User natlee
    port 1994
    Compression yes

Host pi-reversed
    HostName 192.168.1.87
    User pi
    port 22
    Compression yes
    ProxyCommand ssh -W %h:%p reverse-tunnel-inside-bridge
```

第一個 host，我們可以連到外網 server A，也就是 port 爲`1984`的那位

第二個 host，我們可以連到內網裝置 B，也就是經過反向後，port 爲`1994`的那位

最重要的是第三個 host，裏面這句`ProxyCommand ssh -W %h:%p reverse-tunnel-inside-bridge`是精髓

意思是透過第二個 host `reverse-tunnel-inside-bridge` 進行 proxy 連到區網的位置 `192.168.1.87`

有了這個快速連線法，我們就可以直接來體驗一下

```bash
❯ ssh pi-reversed
Linux raspberrypi 5.10.17-v7l+ #1421 SMP Thu May 27 14:00:13 BST 2021 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb 10 01:17:55 2022 from 192.168.1.133

pi@raspberrypi:~ $

```

這樣配置之後，我們很簡單就可以直接從外網穿透到其他內部機器上了！

## 結語

---

這篇實驗做滿久的，不過從頭跑一次整個流程之後，我也對打洞有更深一步的認識

打洞的好處很多，像是 vscode 可以使用 remote-ssh 的套件

我們就可以直接選擇`./ssh/config`內的機器進行連線

而且 SFTP 也是走 SSH 通道，我們可以直接在外網做內網檔案的傳輸

甚至我們也可以直接用內網做 proxy 來瀏覽內部才能看的網站

但是，這樣做其實會有些資安風險

從上面的流程，我們可以看出來`反向`這件事是從內部機器主動連到外網機器

也就是如果有心人士拿到內部機器的存取權限的話，外部機器自然不保了

這邊能做的也只有保管好金鑰了

> 2024-08-13 更新

雖說內網可以反向連到外網機器很危險，但我們只要在外網機器SSH的設定檔做一些限制就可以了

```
# /etc/ssh/sshd_config

Match User telepy
AllowTcpForwarding yes
PermitTunnel yes
ForceCommand echo 'This account can only be used for reverse SSH tunneling'
```
我們在`sshd_config`中加入這段設定，就可以限制只有`telepy`這個使用者
然後，我們的外網機器只有這個使用者且這個使用者只能進行tunneling，其他的操作都會被拒絕
這樣就可以避免內網機器拿到外網機器的命令執行權限

這邊可以參考後來我做的一個新專案[Telepy](https://github.com/NatLee/telepy)
這個專案提供SSH通道在Web UI上的管理界面以及基本的使用者管理
甚至還有Web Terminal的功能，可以直接在瀏覽器上操作遠端機器

有興趣的話可以去看看！😎

## Reference

---

[使用 SSH 反向隧道进行内网穿透](http://arondight.me/2016/02/17/%E4%BD%BF%E7%94%A8SSH%E5%8F%8D%E5%90%91%E9%9A%A7%E9%81%93%E8%BF%9B%E8%A1%8C%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/)

[SSH TUNNELLING FOR FUN AND PROFIT: AUTOSSH](https://www.everythingcli.org/ssh-tunnelling-for-fun-and-profit-autossh/)

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E4%BD%BF%E7%94%A8docker-container%E5%BB%BA%E7%AB%8Bssh%E5%8F%8D%E5%90%91%E9%80%9A%E9%81%93%E7%A9%BF%E9%80%8F%E5%85%A7%E7%B6%B2%E9%80%A3%E6%8E%A5%E5%85%A7%E9%83%A8%E8%A3%9D%E7%BD%AE-716f0c049223)
