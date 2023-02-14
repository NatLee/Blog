---
title: Python 多執行緒、多程序及平行套件速度差異
categories: dev
tags:
  - python
  - concurrent
  - multithread
  - multiprocessing
abbrlink: 217733b8
date: 2023-02-14 00:00:00
---

## 前言

曾經有提到[Python 中的多執行緒 (multithread) 及多程序 (multiprocessing)](https://natlee.github.io/Blog/posts/4162982688/)

但當初沒有比較 Python 內較高層的平行化 API（concurrent.futures），所以再賺一篇文章

---

<!--more-->

## 內容

這次我們將會比較 python 內官方套件做平行處理用的 package 們

- threading
- multiprocessing
- concurrent.futures.ProcessPoolExecutor
- concurrent.futures.ThreadPoolExecutor

### 機器規格

> 2021 Macbook Pro M1 Pro 16" 512GB / 32GB

### Baseline 測試

基本上，平行化的測試只要讓運算量大到能夠看到差距即可

所以，我採用下面這個腳本去做評測：

```python
for i in [7, 7, 7, 7, 7]: # 5 times
    s = 0
    for i in range(10**i): # 10000000
        s = s + 1 # sum
```

這邊要注意的是，有些範例會在平行的函數內使用`time.sleep`

但這樣並沒有做運算，只能看出任務被挑選出來的執行順序可能不同，並不是正確的測試方式

### 最終評測結果

- 一般套件（threading & multiprocessing）

|         | Baseline | Multithread | Multiprocessing | Multiprocessing using pool |
| ------- | -------- | ----------- | --------------- | -------------------------- |
| Time(s) | 1.357    | 1.277       | 0.332           | 0.337                      |

- 高層套件（concurrent.futures）

|         | Multiprocessing with executor | Multithreading with executor |
| ------- | ----------------------------- | ---------------------------- |
| Time(s) | 0.317                         | 1.286                        |

從結果可以看出，高層的 API 跟一般的 API 時間上差不了太多

但是使用上來說，高層的 API 還是比較便利

---

## 測試用腳本

如果有興趣可以使用下面這個腳本玩玩看：

```python
import threading
import multiprocessing
import concurrent.futures
import time

test = [7, 7, 7, 7, 7]

def job(x):
    '''
    x = x * x
    time.sleep(5) # Use `time.sleep()` to test effectiveness for paralleling is a wrong example.
    return x
    '''
    s = 0
    for i in range(10**x):
        s = s + 1
    return s

def runBaseLine():
    """Baseline"""
    starttime = time.time()
    for i in test:
        job(i)
    print('Baseline: {:0.3f} seconds'.format(time.time() - starttime))

def runMultithread():
    """Use threading"""
    threads = list()
    starttime = time.time()
    for element in test:
        threads.append(threading.Thread(target=job, args=(element,)))
    for thread in threads:
        thread.start()
    for thread in threads:
        thread.join()
    print('Multithread: {:0.3f} seconds'.format(time.time() - starttime))

def runMultiprocessing():
    """Use multiprocessing"""
    starttime = time.time()
    processes = []
    for i in test:
        p = multiprocessing.Process(target=job, args=(i,))
        processes.append(p)
        p.start()
    for process in processes:
        process.join()
    print('Multiprocessing: {:0.3f} seconds'.format(time.time() - starttime))


def runMultiprocessingUsingPool():
    """Use multiprocessing with pool"""
    starttime = time.time()
    with multiprocessing.Pool() as p:
        p.map(job, test)
    print('Multiprocessing pool: {:0.3f} seconds'.format(time.time() - starttime))


def runMultiprocessingUsingExePool(max_workers=8):
    """Use high-level multiprocessing class"""
    starttime = time.time()
    with concurrent.futures.ProcessPoolExecutor(max_workers=max_workers) as executor:
        results = executor.map(job, test)
    print('Multiprocessing pool executor: {:0.3f} seconds'.format(time.time() - starttime))

def runMultithreadingUsingExePool(max_workers=8):
    """Use high-level multithreading class"""
    starttime = time.time()
    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        results = executor.map(job, test)
    print('Multithreading pool executor: {:0.3f} seconds'.format(time.time() - starttime))

if __name__ == '__main__':

    runBaseLine()
    runMultithread()
    runMultiprocessing()
    runMultiprocessingUsingPool()
    runMultiprocessingUsingExePool()
    runMultithreadingUsingExePool()

```

## Contributor

這邊要感謝[H-Alice](https://github.com/H-Alice)跟我分享這組 API

這篇文章的測試腳本其實在當初那篇比較文發出來一年後就已經寫好了 XD

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/python-%E5%A4%9A%E5%9F%B7%E8%A1%8C%E7%B7%92-%E5%A4%9A%E7%A8%8B%E5%BA%8F%E5%8F%8A%E5%B9%B3%E8%A1%8C%E5%A5%97%E4%BB%B6%E9%80%9F%E5%BA%A6%E5%B7%AE%E7%95%B0-755f889020a3)
