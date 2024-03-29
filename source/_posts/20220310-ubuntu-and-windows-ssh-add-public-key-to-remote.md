---
title: Ubuntu與Windows使用SSH金鑰快速登入的方法
categories: dev
tags:
  - ssh
  - windows
  - ubuntu
  - linux
abbrlink: 5a40e3d8
date: 2022-03-10 11:37:10
---

## 前言

---

我們開發的時候，常常會使用 SSH 來進行遠端其他 Linux 機器

每次登入除了輸入密碼外，其實還可以使用金鑰快速登入

<!--more-->

## 內容

---

假設我們在`~/.ssh/config`內已經設定了一臺機器去進行連線

```
Host test
    HostName 192.168.1.87
    User testuser
    Port 2468
    Compression yes
```

這樣每次登入那臺機器都可以使用簡短的指令

```bash
ssh test
```

但是輸入密碼從來都是一件惱人的事

所以我們可以使用`ssh-keygen`去建立公私鑰對，再把公鑰放到遠端機器上

```bash
ssh-keygen -t rsa -b 8192 -C "nat-key"
```

- Ubuntu

  以 Ubuntu 來說，有個工具叫做`ssh-copy-id`，根據上面的設定，我們只要執行下面指令並輸入密碼就可以完成公鑰的傳送

  ```bash
  ssh-copy-id test
  ```

- Windows

  在 Windows 上面就沒有那麼方便，我們必須直接把我們的公鑰複製到遠端機器上，這樣才能讓它認得我們的機器

  ```powershell
  type $env:USERPROFILE\.ssh\id_rsa.pub | ssh test "cat >> .ssh/authorized_keys"
  ```

公鑰成功傳送過去後，我們就可以不用密碼，直接使用`ssh test`來進行遠端機器的連線了！

## Reference

---

[3 Steps to Perform SSH Login Without Password Using ssh-keygen & ssh-copy-id](https://www.thegeekstuff.com/2008/11/3-steps-to-perform-ssh-login-without-password-using-ssh-keygen-ssh-copy-id/)

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/ubuntu-%E8%88%87-windows-%E4%BD%BF%E7%94%A8-ssh-%E9%87%91%E9%91%B0%E5%BF%AB%E9%80%9F%E7%99%BB%E5%85%A5%E7%9A%84%E6%96%B9%E6%B3%95-823a8b0211e3)
