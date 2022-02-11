---
title: Matlab 取代語法
categories: develop
tags:
  - matlab
  - code
abbrlink: 419938887
date: 2015-12-23 00:00:00
---

## 前言
----------
一直忘記，記錄一下。

<!--more-->

## 方法
----------

把矩陣A中**NaN**的值取代為**0**
```r
    A(isnan(A)==1) = 0
```

把矩陣A中**0**的值取代為**NaN**
```r
    A(A==0) = NaN
```

把矩陣A中**大於10**的值用**3**來取代
```r
    A((A>10)==1) = 3;
```


## Reference
----------
[Matlab 取代的語法](https://goo.gl/AVsQmr)
