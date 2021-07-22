---
title: Python中的多執行緒(multithread)及多程序(multiprocessing)
date: 2019-08-05
categories: code
tags: [python, multithread, multiprocessing]
---

![](http://i.imgur.com/7w9laRD.jpg)
<center>*ソードアート・オンライン*</center>

## 前言
----------

在很多時候，我們想要加快程式的速度時，除了花錢買更好的硬體外，也會考慮將手邊重複的工作進行平行處理

在Python中，平行處理的方式已經被包裝成特定模組

但是，多執行緒(multithread)與多程序(multiprocessing)還是有所不同

<!--more-->

## 內容
----------

理論上，以前學過的作業系統中對於執行緒(thread)與程序(process)各有解釋

### 執行緒(Thread)

- 同一顆CPU執行

- 所有執行緒共享記憶體

- 一個執行緒被中斷會導致集體死亡

### 程序(Process)

- 每個程序擁有整份程式碼

- 各個程序有獨立的記憶體空間

- 可以多顆CPU運行


在Python中，這邊比較需要注意到的問題是CPU。

Python本身對於執行緒排程進行優化，所以並不可完全適用理論上的結果。

假設今天我們有一函數`job`如下：

```python
def job(x):
    x = x * x
    time.sleep(5)
    if x == 0:
        'zero'
    elif x%2 == 0:
        'even'
    else:
        'odd'
    return
```

我們執行下面函數來得到單一thread單一process的baseline

```python
def runBaseLine():
    starttime = time.time()
    for i in range(0,10):
        job(i)
    print('Baseline took {} seconds'.format(time.time() - starttime))
```

它的執行結果是：

```
Baseline took 50.02211403846741 seconds
```

這個程式花了50秒的時間來完成

接下來，我們要進行多執行緒與多程序的結果比較


### 比較結果

![](http://i.imgur.com/wbFo504.png)
<center>*ソードアート・オンライン*</center>

- **Multithread**

我們以下程式使用多執行緒來執行任務

```python
def runMultithread():
    threads = list()
    starttime = time.time()
    for element in range(0,10):
        threads.append(threading.Thread(target=job, args=(element,)))
    for thread in threads:    
        thread.start()
    for thread in threads:
        thread.join()
    print('Multithread took {} seconds'.format(time.time() - starttime))
```

多執行緒的結果如下

```
Multithread took 5.0071680545806885 seconds
```

- **Multiprocessing**

我們以下程式使用多程序來執行任務

```python
def runMultiprocessing():
    starttime = time.time()
    processes = list()
    for i in range(0,10):
        p = multiprocessing.Process(target=job, args=(i,))
        processes.append(p)
        p.start()
    for process in processes:
        process.join()
    print('Multiprocessing took {} seconds'.format(time.time() - starttime))
```

多程序的結果如下

```
Multiprocessing took 5.031498193740845 seconds
```


- **Multiprocessing using pool**

我們以下程式使用池(pool)來分配多程序來執行任務

```python
def runMultiprocessingUsingPool():
    test = list(range(0,10))
    starttime = time.time()
    with multiprocessing.Pool() as p:
        p.map(job, test)
    print('Multiprocessing using pool took {} seconds'.format(time.time() - starttime))
```

分配方法的結果如下

```
Multiprocessing using pool took 15.129114151000977 seconds
```

我們把結果統整成表格

|         	| Baseline 	| Multithread 	| Multiprocessing 	| Multiprocessing using pool 	|
|---------	|----------	|-------------	|-----------------	|---------------------------	|
| Time(s) 	| 50.022   	| 5.007       	| 5.031           	| 15.129                     	|

由表格中，我們可以發現在Python中多執行緒不一定是只使用單顆CPU執行，因為多程序與多執行緒結果差不多

但令人意外的是使用pool來分配的多程序執行相比一般多程序卻會花較多時間


## Reference
----------

[Using Multiprocessing to Make Python Code Faster](https://medium.com/@urban_institute/using-multiprocessing-to-make-python-code-faster-23ea5ef996ba)

