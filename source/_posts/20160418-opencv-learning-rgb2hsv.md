---
title: OpenCV學習筆記－RGB轉HSV
date: 2016-04-18
categories: study
tags: [opencv, code]
---

![](http://i.imgur.com/HdGGqIw.jpg)
<center>*最弱無敗機龍*</center>

## 前言
----------
RGB轉HSV，並分離成單通道R/G/B與H/S/V

<!--more-->

## 方法
----------

首先，先回憶上次開圖的方法
```c++
	#include "opencv2/highgui/highgui.hpp"
	#include <iostream>
	using namespace cv;
	using namespace std;
	int main() {
		IplImage* img = cvLoadImage("test.png", 1); //1表示三通道
		namedWindow("test",WINDOW_AUTOSIZE);
		cvShowImage("test", img);
		cvWaitKey(0);
		return 0;
	}
```

接下來在其中加入兩行程式碼
```c++
	//以img的大小製作一個空白的空間
	IplImage* hsv = cvCreateImage(cvGetSize(img), 8, 3);
	//把img從RGB轉成HSV
	cvCvtColor(img, hsv, CV_BGR2HSV);
```
> **cvCreateImage(size,depth,channels)**
> **size**代表圖像大小
> **depth**為圖像元素的位深度,**IPL_DEPTH_8U**或**8**代表無符號8位整型
> **channels**代表圖像通道數量

這樣就成功以內建函式的方式轉換了原圖

![](http://i.imgur.com/l6lihNQ.png)

詳細轉換的方式可參考OpenCV文件
[點我到OpenCV](http://docs.opencv.org/2.4/modules/imgproc/doc/miscellaneous_transformations.html?highlight=cvtcolor#cvtcolor)


再來為了分離三個通道成R/G/B得先宣告存放空間並把img分離
```c++
	IplImage* Bimg=cvCreateImage(cvGetSize(img),8,1);
	IplImage* Gimg=cvCreateImage(cvGetSize(img),8,1);
	IplImage* Rimg=cvCreateImage(cvGetSize(img),8,1);
	cvSplit(img, Bimg, Gimg, Rimg, 0);//複製各個通道到三個空間
	//目標圖像必須與來源圖像在大小和資料型態上一樣
	//輸入多通道，輸出為B，G，R單通道
```

但這時要注意，圖片因為單通道所以還是**灰階的**
要再宣告三個三通道的圖片空間
```c++
	IplImage* pImg1=cvCreateImage(cvGetSize(img),8,3);
	IplImage* pImg2=cvCreateImage(cvGetSize(img),8,3);
	IplImage* pImg3=cvCreateImage(cvGetSize(img),8,3);
```
並且把空間初始化
```c++
	cvSetZero(pImg1);
	cvSetZero(pImg2);
	cvSetZero(pImg3);
```
此時，把三個通道合成後可以得到一張完整的圖(R/G/B)
```c++
	//注意順序是BGR
	cvMerge(Bimg,0,0,0,pImg3);
	cvMerge(0,Gimg,0,0,pImg2);
	cvMerge(0,0,Rimg,0,pImg1);
	//將三個通道合成
	//輸入参數為B，G，R單通道，最後一個為輸出
```
最後，開個視窗把圖片輸出就成了
注意這邊是輸出**pImg1、pImg2、pImg3**
```c++
	namedWindow("B", WINDOW_AUTOSIZE);
	cvShowImage("B", pImg3);
	namedWindow("G", WINDOW_AUTOSIZE);
	cvShowImage("G", pImg2);
	namedWindow("R", WINDOW_AUTOSIZE);
	cvShowImage("R", pImg1);
```
![](http://i.imgur.com/AiAn03X.png)

為了分離H/S/V，我們必須也做同樣的事
```c++
	//利用一開始轉換好的HSV宣告三個單通道圖片空間
	IplImage* Himg = cvCreateImage(cvGetSize(hsv), 8, 1);
	IplImage* Simg = cvCreateImage(cvGetSize(hsv), 8, 1);
	IplImage* Vimg = cvCreateImage(cvGetSize(hsv), 8, 1);

	//宣告三個三通道圖片空間
	IplImage* HSV1 = cvCreateImage(cvGetSize(hsv), 8, 3);
	IplImage* HSV2 = cvCreateImage(cvGetSize(hsv), 8, 3);
	IplImage* HSV3 = cvCreateImage(cvGetSize(hsv), 8, 3);

	//把hsv複製到三個空間
	cvSplit(hsv, Vimg, Simg, Himg, 0);

	//把圖片空間初始化
	cvSetZero(HSV1);
	cvSetZero(HSV2);
	cvSetZero(HSV3);

	//把三個通道合成，注意順序是VSH
	cvMerge(Vimg, 0, 0, 0, HSV3);
	cvMerge(0, Simg, 0, 0, HSV2);
	cvMerge(0, 0, Himg, 0, HSV1);
```
輸出後，我們可以得到三張圖，分別是H/S/V

![](http://i.imgur.com/e3458Z0.png)


試著一次把所有圖都輸出

![](http://i.imgur.com/Mql1XuL.png)


整個程式的程式碼可以參考如下
```c++
	#include "opencv2/highgui/highgui.hpp"
	#include <iostream>

	using namespace cv;
	using namespace std;

	int main() {
		IplImage* img = cvLoadImage("test.png", 1); //1表示三通道

		//以img的大小製作一個空白的空間
		IplImage* hsv = cvCreateImage(cvGetSize(img), 8, 3);
		//把img從RGB轉成HSV
		cvCvtColor(img, hsv, CV_BGR2HSV);

		//轉R/G/B單通道-----------------------
		IplImage* Bimg = cvCreateImage(cvGetSize(img), 8, 1);//B單通道灰階
		IplImage* Gimg = cvCreateImage(cvGetSize(img), 8, 1);//G單通道灰階
		IplImage* Rimg = cvCreateImage(cvGetSize(img), 8, 1);//R單通道灰階
		cvSplit(img, Bimg, Gimg, Rimg, 0);

		IplImage* pImg1 = cvCreateImage(cvGetSize(img), 8, 3);
		IplImage* pImg2 = cvCreateImage(cvGetSize(img), 8, 3);
		IplImage* pImg3 = cvCreateImage(cvGetSize(img), 8, 3);
		cvSetZero(pImg1);
		cvSetZero(pImg2);
		cvSetZero(pImg3);
		cvMerge(Bimg, 0, 0, 0, pImg3);
		cvMerge(0, Gimg, 0, 0, pImg2);
		cvMerge(0, 0, Rimg, 0, pImg1);
		//----------------------------------------

		//轉H/S/V單通道-----------------------
		IplImage* Himg = cvCreateImage(cvGetSize(hsv), 8, 1);
		IplImage* Simg = cvCreateImage(cvGetSize(hsv), 8, 1);
		IplImage* Vimg = cvCreateImage(cvGetSize(hsv), 8, 1);
		IplImage* HSV1 = cvCreateImage(cvGetSize(hsv), 8, 3);
		IplImage* HSV2 = cvCreateImage(cvGetSize(hsv), 8, 3);
		IplImage* HSV3 = cvCreateImage(cvGetSize(hsv), 8, 3);
		cvSplit(hsv, Vimg, Simg, Himg, 0);

		cvSetZero(HSV1);
		cvSetZero(HSV2);
		cvSetZero(HSV3);
		cvMerge(Vimg, 0, 0, 0, HSV3);
		cvMerge(0, Simg, 0, 0, HSV2);
		cvMerge(0, 0, Himg, 0, HSV1);
		//----------------------------------------

		cvNamedWindow("HSV", WINDOW_AUTOSIZE);
		cvShowImage("HSV", hsv);

		namedWindow("H", WINDOW_AUTOSIZE);
		cvShowImage("H", HSV1);
		namedWindow("S", WINDOW_AUTOSIZE);
		cvShowImage("S", HSV2);
		namedWindow("V", WINDOW_AUTOSIZE);
		cvShowImage("V", HSV3);


		cvNamedWindow("RGB", WINDOW_AUTOSIZE);
		cvShowImage("RGB", img);

		namedWindow("R", WINDOW_AUTOSIZE);
		cvShowImage("R", pImg1);
		namedWindow("G", WINDOW_AUTOSIZE);
		cvShowImage("G", pImg2);
		namedWindow("B", WINDOW_AUTOSIZE);
		cvShowImage("B", pImg3);

		cvWaitKey(0);

		return 0;
	}
```






## Reference
----------
[[OpenCV] 轉換色彩空間 (Transform Color Space)](https://cg2010studio.com/2012/10/09/opencv-%E8%BD%89%E6%8F%9B%E8%89%B2%E5%BD%A9%E7%A9%BA%E9%96%93-transform-color-space/)
[基於Opencv下RGB圖像轉HSV，並分離成單通道R/G/B與H/S/V](http://fanli7.net/a/bianchengyuyan/C__/20140502/498266.html)
