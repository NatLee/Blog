title: 修改Windows的登入密碼
date: 2016-01-31
categories: Tech
tags: [Windows]
---

![更重要的是努力的話總會有回報的](http://i.imgur.com/S6qH6xo.jpg)
<center>*紅殼的潘朵拉*</center>

努力的話，真的就有回報嗎？

前言
--
 ----------
常常有人忘記密碼或電腦被鎖(?)，然後就會上網找攻略。
不過大部份攻略又是外文，這邊就做個小整理。


<!--more-->

## 方法 ##
----------

<font color="red">**※點擊圖片可以放大**</font>

### Windows 7 ###
----------

<font color="blue">**圖形化界面的置換修改法**</font>

1. 插入**Windows7**安裝光碟
2. 進入安裝畫面後，按下**下一步**再按**修復您的電腦**
3. 按下**載入驅動程式**
![](http://i.imgur.com/HBGt2Df.png)
4. 進到`windows\system32`目錄底下
5. 把`Magnify.exe`改名成`Magnify1.exe`
![](http://i.imgur.com/4nk5LR3.png)
6. 把`cmd.exe`改名成`Magnify.exe`
![](http://i.imgur.com/YQQRNz5.png)
![](http://i.imgur.com/ZC8gFsL.png)
7. 重新啟動電腦
8. 進入登入畫面按左下角協助工具選擇**放大鏡**
![](http://i.imgur.com/j5JlQCC.png)
9. 輸入`net user 你的帳戶 123`後按Enter把密碼改成**123**
![](http://i.imgur.com/M7R14C3.png)
10. 重新啟動進到光碟裡把**放大鏡跟cmd**改回原本名字


<font color="blue">**指令置換修改法**</font>

1. 插入**Windows7**安裝光碟
2. 進入安裝畫面後，按下**Shift+F10**
3. 輸入`cd\`後按Enter跳至系統槽（可能C或D）
4. 輸入`cd windows\system32`後按Enter跳到目錄底下
5. 輸入`ren Magnify.exe Magnify1.exe`後按Enter把放大鏡改名
6. 輸入`ren cmd.exe Magnify.exe`後按Enter把cmd改成放大鏡
7. 關閉Command Window後重新啟動電腦
8. 進入登入畫面按左下角協助工具選擇**放大鏡**
9. 輸入`net user 你的帳戶 123`後按Enter把密碼改成**123**
11. 輸入`cd\`後按Enter跳至系統槽（可能C或D）
10. 重新啟動進到光碟後按下**Shift+F10**
12. 輸入`cd windows\system32`後按Enter跳到目錄底下
13. 輸入`ren Magnify.exe cmd.exe`後按Enter把cmd改回來
14. 輸入`ren Magnify1.exe Magnify.exe`後按Enter把放大鏡改回來
15. 重新啟動電腦即完成，密碼為**123**

精隨其實就是把放大鏡跟CMD互換
所以任何一個作業系統安裝光碟都可以修改密碼
![當時我們不知道未來有多殘酷的試煉](http://i.imgur.com/jDFxVac.jpg)
<center>*房東妹子青春期*</center>

### Windows 8/8.1/10 ###
----------

Windows8/8.1/10的方法都一樣，所以直接放在一起。

<font color="blue">**指令置換修改法**</font>

1. 插入**Windows10**安裝光碟
![](http://i.imgur.com/oAoqGMO.png)
2. 進入安裝畫面後，按下**Shift+F10**
![](http://i.imgur.com/Sa2Nrj8.png)
3. 輸入以下指令後按Enter把**utilman.exe**改名加尾綴**.bak**
`move d:\windows\system32\utilman.exe d:\windows\system32\utilman.exe.bak`
4. 輸入以下指令後按Enter把**cmd.exe**改名成**utilman.exe**
`copy d:\windows\system32\cmd.exe d:\windows\system32\utilman.exe`
![](http://i.imgur.com/gwGOzrs.png)
 > 圖中前兩次找不到路徑是system32少打32
 > 第三次找不到是因為系統槽在**D槽**
 > 所以這邊要注意有沒有錯字

5. 關閉Command Window後重新啟動電腦
6. 進入登入畫面按**協助工具**（左下角或右下角）
7. 輸入`net user 你的帳戶 123`後按Enter把密碼改成**123**
![](http://i.imgur.com/HJ5MYM4.png)
8. 重新啟動進到光碟後按下**Shift+F10**
9. 依序輸入以下指令把檔案換回來
`move d:\windows\system32\utilman.exe d:\windows\system32\cmd.exe`
`move d:\windows\system32\utilman.exe.bak d:\windows\system32\utilman.exe`
10. 重新啟動電腦即完成，密碼為**123**

這邊要注意一下，這個方法僅限系統管理員Account非Microsoft帳號
如果你的系統管理員身份是M$帳號的話
請在**第七步**時把指令改成`net user 你要新增的帳戶 /add`
這樣就可以進到有管理權限的帳戶來改掉原本進不去的帳戶密碼了

# 後記 #
----------
![我從來都沒想過這點](http://i.imgur.com/dfSNJi6.jpg)
<center>*房東妹子青春期*</center>

Win10換完檔案後，其實可以用輸入`wpeutil reboot`重啟
![](http://i.imgur.com/tukE2wl.png)

不過改密碼的方法一樣都是**置換法**，總之能開cmd的話萬事都OK。
感謝收看

# Reference #
----------
[Reset a Windows 10 password](https://4sysops.com/archives/reset-a-windows-10-password/)
