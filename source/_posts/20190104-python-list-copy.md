---
title: Python中List的Copy反直覺踩坑
date: 2019-01-04
categories: develop
tags: [python, code, develop]
---

![](https://i.imgur.com/7mrHp3H.jpg)
<center>*佐賀偶像是傳奇*</center>

## 前言
----------

你以為的以為，跟我以為的以為
坑！都是坑！滿滿的坑！

<!--more-->

## 掰個例子
----------

身為一個肥宅在Python中處理資料是常見的事

舉個簡單土砲的 `.csv` 檔讀取例子（本文皆使用Python3.6， `.csv` 檔為逗號分隔）

```python
def csvLoader(csvPath:str) -> list:
    """ load csv to list format data """
    # Windows系統用Excel存的 .csv 大部分都是 Big5 格式，柳暗花明又一坑 : (
    with open(csvPath, 'r', encoding='utf-8') as csv:
        parseList = list() # store all data
        columnNames = csv.readline().replace('\n', '').split(',')[1:] # load column names in first row and ignore first column which is id

        rowData = dict() # use rowData to store all data in .csv file

        for line in csv.readlines():
            raw = line.replace('\n', '').split(',') # here split by ','
            for i, columnName in enumerate(columnNames):
                rowData[columnName] = raw[i+1] # cause we ignore first column, so we need '+1'.
            parseList.append(rowData)
    return parseList
```

各位觀眾看看上面的 code，看起來似乎.....沒有問題？

## 夜路走多踩到坑
----------

如何分辯程式執行是否有問題
不如我們直接拿個 .csv 檔丟進去做讀檔測試

以下是我們今天測試檔案 test.csv 的格式

|   	| name   	| sex    	| phone      	| birthday 	|
|:-:	|--------	|--------	|------------	|----------	|
| 1 	| Honoka 	| female 	| 0912345678 	| 8/3      	|
| 2 	| Chika  	| female 	| 8765432190 	| 8/1      	|
| 3 	| Ereka  	| none   	| 0000000000 	| 1/1      	|


```python

>>> csvLoader('test.csv')
[
    {'name': 'Ereka', 'sex': 'none', 'phone': '0000000000', 'birthday': '1月1日'},
    {'name': 'Ereka', 'sex': 'none', 'phone': '0000000000', 'birthday': '1月1日'},
    {'name': 'Ereka', 'sex': 'none', 'phone': '0000000000', 'birthday': '1月1日'}
]

```

太神奇了傑克，竟然全部都是 Ereka ！

所以問題到底出在哪...?


## 以為的以為
----------

假設這世界上只有兩個人，一個是 a ，另一個是它的複製人

```python
 a = [ 1,2,3 ]
 b = a
```

這時， b 等於 [1, 2, 3]

但今天我們把 a 進行擴充

```python
a.append(4)
```

a 會變成 [1, 2, 3, 4]，而 b ...

```python
>>> b
[1, 2, 3, 4]
```

b 也變成 [1, 2, 3, 4] 了！
於是我們檢查一下它們的 id

```python
>>> id(a)
4340014344
>>> id(b)
4340014344
```

賓果，id 一樣
但如果今天我們用 copy 的話...

```python
>>> c = a.copy()
>>> c
[1, 2, 3, 4]
>>> a.append(5)
>>> a
[1, 2, 3, 4, 5]
>>> c
[1, 2, 3, 4]
```
Wow！ c 成為全新的個體了！
我們檢查一下它的 id

```python
>>> id(c)
4339725448
```

4339725448 不等於 4340014344
果然是不同的東西了！

## 真相大白
----------

我說那個...

一開始的程式到底錯在哪呢？

```python
parseList = list()
rowData = dict() # use rowData to store all data in .csv file
for line in csv.readlines():
    raw = line.replace('\n', '').split(',') # here split by ','
    for i, columnName in enumerate(columnNames):
        rowData[columnName] = raw[i+1] # cause we ignore first column, so we need '+1'.
    parseList.append(rowData)
```

有看出來了嗎？
我們把它換成另外一個更簡單看清楚的程式

```python
p = list()
a = dict()
for col in cols:
    for i, row in enumerate(rows):
        a[i] = row
    p.append(a)
```

再...再更簡單一點

```python
p = list()
a = list()
for i in range(10):
    a.append(i)
    p.append(a)
```

這樣子最終的輸出會如下

```python
>>> a
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> p
[[0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]]
```

也就是說， p 這個List內所有元素的 id 都一樣
換句話說， p append的都是記憶體位置中同樣的List
所以才會產生這樣的結果

我們回到最一開始的例子

```python
def csvLoader(csvPath:str) -> list:
    """ load csv to list format data """
    # Windows系統用Excel存的 .csv 大部分都是 Big5 格式，柳暗花明又一坑 : (
    with open(csvPath, 'r', encoding='utf-8') as csv:
        parseList = list() # store all data
        columnNames = csv.readline().replace('\n', '').split(',')[1:] # load column names in first row and ignore first column which is id
        for line in csv.readlines():
            rowData = dict() # use rowData to store all data in .csv file
            raw = line.replace('\n', '').split(',') # here split by ','
            for i, columnName in enumerate(columnNames):
                rowData[columnName] = raw[i+1] # cause we ignore first column, so we need '+1'.
            parseList.append(rowData)
    return parseList
```

我們把 rowData 放進 for 迴圈內，每次都重新建立一個全新的 dict
當然， id 也不同

parseList append的 rowData id 都不一樣
最後也就不會大家都長一樣了

## 總結
----------

感恩讚嘆God Alice <(_ _)>
