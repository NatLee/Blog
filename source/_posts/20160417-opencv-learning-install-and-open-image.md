---
title: OpenCV學習筆記－安裝與開圖
categories: study
tags:
  - opencv
  - code
  - c++
abbrlink: 3012259224
date: 2016-04-17 00:00:00
---

![](http://i.imgur.com/aEGYK6p.jpg)
<center>*少年女僕*</center>

## 前言
----------

OpenCV的安裝與環境設置

<!--more-->

## 方法
----------

一開始先到OpenCV官網下載安裝
[點我到OpenCV官網](http://opencv.org/ "OpenCV官網")
通常都安裝在C槽中，安裝完後到系統設定環境變數

	C:\OpenCV3.1\build\x64\vc12\bin
	C:\OpenCV3.1\build

設定完後，重開機開啟VS2013
開新專案後，到右邊找到**屬性管理員**
對**Debug | Win32**按右鍵**新增新的專案屬性工作表**
接下來對它按右鍵**屬性**並找到**VC++目錄**的**Include目錄**並編輯它
![](http://i.imgur.com/BIkxMqv.png)
接著看到**程式庫目錄**，一樣編輯它
![](http://i.imgur.com/h5vWUaF.png)
再來看左邊清單找到**連結器**
![](http://i.imgur.com/xnnYSBz.png)
點擊它，再點**輸入**後一樣編輯**其他相依性**
![](http://i.imgur.com/zP70e7h.png)
這樣就完成專案的設定了
然後來寫第一個OpenCV的程式
```c++
	#include "opencv2/highgui/highgui.hpp"//引入OpenCV的功能
	using namespace cv;//使用cv名稱
	int main(){
		IplImage* img = cvLoadImage("test.jpg");//宣告一個圖片資源
		namedWindow("test");//命名視窗
		cvShowImage("test", img);//以命名的視窗開起圖片
		cvWaitKey(0);//等待
		return 0;
	}
```
![](http://i.imgur.com/S6SHo7W.png)

當然用Mat的方式也可以開圖
```c++
	Mat img = imread("test.jpg", CV_LOAD_IMAGE_COLOR);
```
但是秀圖的方式要改成**imshow**
```c++
	namedWindow( "test");
	imshow( "test", image );
```
而IplImage與Mat的格式也能互轉

**IplImage -> Mat**
```c++
	IplImage* img = cvLoadImage("test.jpg", 1);
	Mat img2(img);
```

**Mat -> IplImage**
```c++
	Mat img = imread("test.jpg", 1); //用Mat型別讀取圖片;
	IplImage* img2;  //宣告IplImage的變數
	img2=&IplImage(img) ; //把Mat位置傳給img2
```


## Reference
----------
[OpenCV学习笔记（一）——安装配置、第一个程序](http://blog.csdn.net/yang_xian521/article/details/6894228)
[OpenCV学习笔记（三）——Mat，图像的新容器](http://blog.csdn.net/yang_xian521/article/details/6894716)
