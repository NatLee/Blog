---
title: 使用OpenSSH來生成公私鑰
categories: dev
tags:
  - openssh
  - cryptograph
  - security
  - RSA
  - ECC
abbrlink: 43b18a36
date: 2022-07-01 00:00:00
---

## 前言

---

記錄一下，免得有時候忘記

畢竟鑰匙通常只會產生一次

<!--more-->

## 內容

---

### 建立金鑰對

使用指令可以直接生成 Key

```bash
ssh-keygen
```

填入 key 要存的位置，例如我填入`test_rsa`

它就會在當前目錄產生兩個檔案

```bash
(base) PS C:\Users\NatLee> ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\NatLee/.ssh/id_rsa): test_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in test_rsa.
Your public key has been saved in test_rsa.pub.
The key fingerprint is:
SHA256:pCx0UfAKJhPGD7W5rZ/O1zE9fFjenAGTdFI7GS7I4+w natlee@Nat-PC
The key's randomart image is:
+---[RSA 3072]----+
| .o.. oo.   .oo+ |
| .o. o o  . .++ +|
|  oo* . o  + .o= |
|   =.* +  o . o..|
|    o = S  = + oo|
|     o    + = o.o|
|    .    . E o   |
|     o .. .      |
|     .=.         |
+----[SHA256]-----+
```

一個是公鑰帶有`.pub`，另一個則是私鑰

```bash
-a---          2022/8/1  下午 10:24     2.54KB test_rsa
-a---          2022/8/1  下午 10:24      568   test_rsa.pub
```

其他自定義的生成方法，範例如下

```bash
ssh-keygen -t rsa -b 4096
ssh-keygen -t dsa
ssh-keygen -t ecdsa -b 521
ssh-keygen -t ed25519
```

使用`-t`可以指令加密類型，`-b`則是 bit 長度

如果想要在建立鑰匙時直接指定名稱，可以使用`-f`

```bash
ssh-keygen -f ./test -t ecdsa -b 521
```

這樣就會在當前目錄建立名稱爲`test`的金鑰對

```bash
-a---          2022/8/1  下午 10:30      736   test
-a---          2022/8/1  下午 10:30      268   test.pub
```

### 複製公鑰到目標機器上

使用以下指令可以複製`指定`金鑰到目標機器

```bash
ssh-copy-id -i ~/.ssh/tatu-key-ecdsa user@host
```

這邊的`-i`是指定要複製的金鑰，`tatu-key-ecdsa`是指定的金鑰名稱

但是這個指令 windows 上面沒有

先前的文章有提到如何在透過 powershell 指令的方式將金鑰複製到目標機器

### 其他補充

這邊是節錄參考的內容，裏面詳細說明每個 flag 的用途

Here's a summary of commonly used options to the keygen tool:

- **b** “Bits” This option specifies the number of bits in the key. The regulations that govern the use case for SSH may require a specific key length to be used. In general, 2048 bits is considered to be sufficient for RSA keys.
- **e** “Export” This option allows reformatting of existing keys between the OpenSSH key file format and the format documented in **[RFC 4716](https://tools.ietf.org/html/rfc4716)**, “SSH Public Key File Format”.
- **p** “Change the passphrase” This option allows changing the passphrase of a private key file with **`[-P old_passphrase]`** and **`[-N new_passphrase]`**, **`[-f keyfile]`**.
- **t** “Type” This option specifies the type of key to be created. Commonly used values are: **rsa** for **[RSA](<https://en.wikipedia.org/wiki/RSA_(cryptosystem)>)** keys **dsa** for **[DSA](https://en.wikipedia.org/wiki/Digital_Signature_Algorithm)** keys **ecdsa** for **[elliptic curve DSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)** keys
- **i** "Input" When *ssh-keygen* is required to access an existing key, this option designates the file.
- **f** "File" Specifies name of the file in which to store the created key.
- **N** "New" Provides a new passphrase for the key.
- **P** "Passphrase" Provides the (old) passphrase when reading a key.
- **c** "Comment" Changes the comment for a keyfile.
- **p** Change the passphrase of a private key file.
- **q** Silence ssh-keygen.
- **v** Verbose mode.
- **l** "Fingerprint" Print the fingerprint of the specified public key.
- **B** "Bubble babble" Shows a "bubble babble" (Tectia format) fingerprint of a keyfile.
- **F** Search for a specified hostname in a known_hosts file.
- **R** Remove all keys belonging to a hostname from a known_hosts file.
- **y** Read a private OpenSSH format file and print an OpenSSH public key to stdout.

This only listed the most commonly used options. For full usage, including the more exotic and special-purpose options, use the `man ssh-keygen` command.

## Reference

---

- [How to use ssh-keygen to generate a new SSH key](https://www.ssh.com/academy/ssh/keygen)
