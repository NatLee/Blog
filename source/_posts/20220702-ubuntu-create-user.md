---
title: 在Ubuntu系統建立使用者
categories: dev
tags:
  - ubuntu
  - linux
abbrlink: 7a79e2e1
date: 2022-07-02 00:00:00
---

## 前言

---

在 Ubuntu 上建立一般使用者並將使用者加入`sudo`群組

<!--more-->

## 內容

---

以下 username 通通以`natlee`作爲範例，如果要使用請自行更改

### 步驟

1. 使用`adduser`創建使用者，並輸入使用者密碼

   ```bash
   sudo adduser natlee
   ```

2. 將使用者加入 sudo group

   ```bash
   sudo adduser natlee sudo
   ```

   或是使用

   ```bash
   sudo usermod -aG sudo natlee
   ```

### 其他建立使用者的方法

- 使用`--gecos`去避開互動式的輸入並使用`--disabled-password`禁用密碼

  ```bash
  adduser --disabled-password --gecos "" natlee
  ```

- 使用`useradd`指令建立使用者

  ```bash
  useradd -m -d /home/natlee -s /bin/bash natlee
  ```

  使用 flag `-d`去指定使用者 home 目錄位置，並用`-s`去指定使用者的預設 shell

### 讓使用者可以使用金鑰登入

不管是用哪種方法建立使用者

我們只要在新使用者的 home 目錄下去建立一個`.ssh`目錄

```bash
mkdir /home/natlee/.ssh
chown natlee:natlee /home/natlee/.ssh
chmod 700 /home/natlee/.ssh
```

並在其中建立一個`.ssh/authorized_keys`檔案，把要連入這個機器的公鑰放進去

就能用這個公鑰的私鑰來作爲新使用者登入這臺機器了

## Reference

---

- [Run adduser non-interactively](https://askubuntu.com/questions/94060/run-adduser-non-interactively)
- [Add a user without password but with SSH and public key](https://unix.stackexchange.com/questions/210228/add-a-user-without-password-but-with-ssh-and-public-key/210232#210232)
