---
title: MIUI7中WSM/XPOSED Framwork無法使用問題
categories: android
tags:
  - android
  - miui
  - wsm
  - xiaomi
abbrlink: 1497250561
date: 2015-12-20 00:00:00
---

## 前言
----------

<font color="red">※功能於最新版已修復</font>

相信很多人把MIUI系統從6更新到7時，發現WSM不能使用了。 這個問題跟系統檔案的**app_process**和**libandroid_runtime.so**有關。

<!-- more -->

## 步驟說明

--------------------------------------------------------------------------------

[新方法按我](http://en.miui.com/thread-78894-1-1.html)

<font color="red">※以下為舊方法</font>

> <font color="RED">WSM/Xposed is not (yet) compatible with Android SDK version 19 or your processor architecture (armeabi-v7a).</font>

SDK對應的版本為： SDK22 = 5.1 SDK21 = 5.0 SDK19 = 4.4 (KitKat)

這個修復方法的概念是把新版系統檔案替換成舊版的檔案。

1. 用**Root Explorer**找到

  ```
    /system/bin/app_process
    /system/lib/libandroid_runtime.so
  ```

2. 把它們備份
3. 下載舊版**app_process**和**libandroid_runtime.so** [點我下載](https://www.androidfilehost.com/?fid=24269982087001158)
4. 複製**/System**內**/bin**及**/lib**的檔案貼上到以下位置

  ```
   app_process → /system/bin/
   libandroid_runtime.so → /system/lib/
  ```

5. 更改兩個檔案的**permissions**

  ```
   app_process (-rwxr-xr-x)
   libandroid_runtime.so (-rw-r--r--)
  ```

6. 重啟並安裝**Xposed/WSM**

## Reference

--------------------------------------------------------------------------------

[XDA [wsm/xposed framework fix for all miui roms]](http://forum.xda-developers.com/xiaomi-mi-3/themes-apps/wsm-xposed-framework-fix-chinese-roms-5-t3221030)
