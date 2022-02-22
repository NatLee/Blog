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
----------

之前一直很想寫反向通道的流程，但過程很難描述

剛好最近在docker上看到好用的openssh-server的image

結果一試就上手，於是就有這篇文章了


<!--more-->


## 步驟說明
----------

> 首先，你需要一個`位於外網且可以透過SSH連線的裝置`
> 
> 如果沒有，請按上一頁

我今天看到一個image [linuxserver/openssh-server](https://hub.docker.com/r/linuxserver/openssh-server)

它把openssh-server的部分都做好了

所以我們可以直接用這個image去run一個container

也就是寫一份docker-compose的文件就搞定了

接下來，我們會分別討論三個裝置上的設定
- 外網裝置 A
- 內網跳板裝置 B
- 透過跳板B連至內網裝置C


### 外網裝置 A

首先，我們先在外網的機器A先佈置能能夠進行被反向連接的server

我們基於docker hub上面官方的文件做修改

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

> 有些版本的docker會出現無法binding volumes的問題，可以先自己用`mkdir`建立資料夾

因爲它的image包好的SSH port是`2222`，那麼我們把外部`1984`對應進去

另外，我們需要多開一個port讓內網裏面做反向的裝置能夠連到這臺server

所以多開了一個port `1994`

> 這邊記得在路由器幫這兩個port做forwarding，建議內外相同port就行了

接着，我們就執行

```
docker-compose up
```

記得，這邊沒有使用`-d`去做背景執行是因爲等等debug比較方便

我們需要修改openssh-server的設定檔`sshd_config`

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

我們就可以放著不管，用背景執行這個container了

#### Extra Step

這邊是額外的連線測試，不做也不影響後面的流程

可以使用local做連線測試

```bash
❯ ssh natlee@localhost -p 1984                          
The authenticity of host '[localhost]:2000 ([::1]:1984)' can't be established.
ECDSA key fingerprint is SHA256:g73WW30kTFqdUvi/mCbIgrW4dnadPiW7ipopK9h6/zk.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[localhost]:1984' (ECDSA) to the list of known hosts.
natlee@localhost: Permission denied (publickey,keyboard-interactive).
```

於是，失敗了，因爲我們只能用key進行連線！

那麼要如何測試連線呢？

我們先找一對金鑰（如果沒有的話，可以用`ssh-keygen`生成）

再把剛找的公鑰（通常預設會在`~/.ssh/id_rsa.pub`）放到`./ssh_setting/.ssh/authorized_keys`內

內容類似長這樣的東西：

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDs13dOFtSQ8OJjIIZiVKWTIUbEvy+ZIcswOiXCb4q4nIK80yNyuDOVcbzQyyS0Q0z0UUTpf+TR5Rs4Ox0B8baffTC0CvFHQhwqpgvz/8d179fmUCyDb9yrdgdhSlN4zkMdw8A2P8SQ+3cc+gIQBvj/jo414kynz4zaoGseFxBOpVKc+dW1Y8m3G4wfBB0QpcyxYZ9vkAhrIYyPrIK9EP7LXTlJyQ7gMadJ4eUkMiBSnqfJYxPcYDIeK/uHYyGgpsJQUaxZ8JIqkD2kdmBPX8NHGA1O2VF2UyiJKuRVJEg/oW7YoCJHs81Gj+bl9HdnBC5CEMWckLPLRtfFkW3u9F6PNto9fs48L1dU8MGm6KAirX2GPydmwRS6yh9i32NT5J70izi28cX1IhJn2PtY75aZAPuU2NspsJ1cy4cbd35tJLB0H0= root@test
```

再用SSH連線一次！

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

但還是老樣子，我們在內網裝置上起一個openssh-server

基於docker hub上面官方的文件做修改

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

改完之後直接run一波

> 有些版本的docker會出現無法binding volumes的問題，可以先自己用`mkdir`建立資料夾

內網的裝置就不用給ports，給了也只有內網電腦能連進去

```bash
docker-compose up
```

run完會看到它的啓動畫面，然後就跟剛剛一樣生一堆文件出來

我們接下來要配置一些文件讓它能夠連到server且斷線後能夠重連

- 建立安裝autossh的腳本

	使用`sudo nano ./ssh_setting/custom-cont-init.d/install.sh`將以下內容貼上

	```bash
	apk add --no-cache autossh
	```

- 建立啓動autossh的腳本

	使用`sudo nano ./ssh_setting/custom-services.d/autossh.sh`將以下內容貼上

	```bash
	#!/bin/bash
	echo "Start AutoSSH"
	autossh \
	 -p 1984 \
	 -M 0 \
	 -NR '*:1994:localhost:2222' \
	 natlee@<EXAMPLE_DOMAIN> \
	 -i ~/.ssh/id_rsa
	```

	這邊請把`<EXAMPLE_DOMAIN>`改成你自己連接外網裝置的domain或IP

接着，我們直接

```bash
docker-compose up
```

然後就噴錯了

```
reverse-tunnel-inside-bridge | Warning: Identity file /root/.ssh/id_rsa not accessible: No such file or directory.
reverse-tunnel-inside-bridge | Host key verification failed.
```

爲什麼它會說無法存取`/root/.ssh/id_rsa`？

因爲它必須要靠key來連到server，但它並沒有key pairs

所以我們得幫他產生一組

我們先別暫停container，而是直接使用這個指令進去container內

```bash
docker exec -it reverse-tunnel-inside-bridge /bin/bash
```

然後使用以下指令產生公私鑰對

```bash
root@nat-tunnel-inside-bridge:/# ssh-key
ssh-keygen   ssh-keyscan
root@nat-tunnel-inside-bridge:/# ssh-keygen
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

我們就可以把公鑰檔案`~/.ssh/id_rsa.pub`的內容複製出來

```bash
root@nat-tunnel-inside-bridge:/# cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC...............bZVysfXr9E= root@nat-tunnel-inside-bridge
```

把公鑰檔案內容複製到我們外網server A的`./ssh_setting/.ssh/authorized_keys`中

再把焦點轉回剛剛的container輸出，還是會有看到錯誤

因爲我們未曾使用這個key連線過

爲了節省麻煩，我們直接進到內網裝置B的container訪問一次server

在B的container內用以下指令訪問一次server，並輸入`yes`去記錄hostname

```bash
root@nat-tunnel-inside-bridge:/# ssh natlee@<EXAMPLE_DOMAIN> -p 1984
The authenticity of host '[EXAMPLE_DOMAIN]:1984 ([XXX.XXX.XXX.XXX]:1984)' can't be established.
ED25519 key fingerprint is SHA256:JNKQF+GwAgca6xPeoz2ROfz6WXe2oo7HepwuemJH58M.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[EXAMPLE_DOMAIN]:1984' (ED25519) to the list of known hosts.
Welcome to OpenSSH Server

nat-tunnel-server:~$
```

這時候看回去container輸出就沒有再跳錯誤了！

再來是安全問題，我們可以再打開目錄中的`./ssh_setting/ssh_host_keys/sshd_config`去把這個設定做更改

```
ChallengeResponseAuthentication no # 安全需求，只開放public key登入
```

到這邊內網裝置B已經算是設定完成了

接下來，我們把自己第三方位置的PC公鑰複製到內網裝置B的`ssh_setting/.ssh/authorized_keys`後，重新執行

```
docker-compose up --force-recreate
```

再來，我們從PC端使用SSH連接server的另外一個port `1994`去做測試

```bash
❯ ssh natlee@<EXAMPLE_DOMAIN> -p 1994
Welcome to OpenSSH Server

nat-tunnel-inside-bridge:~$
```

連到裝置B，也就是成功打洞進到內網了！


### 透過跳板B連至內網裝置C

這邊的裝置C，我使用樹莓派當範例

我們可以利用`~/.ssh/config`去設定SSH的快速連線方式，如下面的例子

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

第一個host，我們可以連到外網server A，也就是port爲`1984`的那位

第二個host，我們可以連到內網裝置B，也就是經過反向後，port爲`1994`的那位

最重要的是第三個host，裏面這句`ProxyCommand ssh -W %h:%p reverse-tunnel-inside-bridge`是精髓

意思是透過第二個host `reverse-tunnel-inside-bridge` 進行proxy連到區網的位置 `192.168.1.87`

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
----------

這篇實驗做滿久的，不過從頭跑一次整個流程之後，我也對打洞有更深一步的認識

打洞的好處很多，像是vscode可以使用remote-ssh的套件

我們就可以直接選擇`./ssh/config`內的機器進行連線

而且SFTP也是走SSH通道，我們可以直接在外網做內網檔案的傳輸

甚至我們也可以直接用內網做proxy來瀏覽內部才能看的網站

但是，這樣做其實會有些資安風險

從上面的流程，我們可以看出來`反向`這件事是從內部機器主動連到外網機器

也就是如果有心人士拿到內部機器的存取權限的話，外部機器自然不保了

這邊能做的也只有保管好金鑰了


## Reference
----------

[使用SSH反向隧道进行内网穿透](http://arondight.me/2016/02/17/%E4%BD%BF%E7%94%A8SSH%E5%8F%8D%E5%90%91%E9%9A%A7%E9%81%93%E8%BF%9B%E8%A1%8C%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/)

[SSH TUNNELLING FOR FUN AND PROFIT: AUTOSSH](https://www.everythingcli.org/ssh-tunnelling-for-fun-and-profit-autossh/)