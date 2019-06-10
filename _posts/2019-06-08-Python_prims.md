---
layout: post  #这个不变
title: "Python学习——用filter求素数" #标题
date: 2019-06-08 14:32 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### 用filter求素数
首先，列出从2开始的所有自然数，构造一个序列：2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, ...

取序列的第一个数2，它一定是素数，然后用2把序列的2的倍数筛掉：
3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, ...

取新序列的第一个数3，它一定是素数，然后用3把序列的3的倍数筛掉：

5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, ...

取新序列的第一个数5，然后用5把序列的5的倍数筛掉：

7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, ...

不断筛下去，就可以得到所有的素数。
```
# 构造一个从3开始的奇数序列，这是一个生成器，并且是一个无限序列
def _odd_iter():
    n = 1
    while True:
        n = n + 2
        yield n  # 3 5 7 9 11 13...


# 筛选函数
def _not_divisible(n):
    return lambda x: x % n > 0  # 进行筛选保留不是n的倍数的数


# 定义一个生成器，不断返回下一个素数
def prims():
    yield 2
    it = _odd_iter()  # 初始序列: 3 5 7 9 11 13...
    while True:
        n = next(it)
        yield n
        it = filter(_not_divisible(n), it)


for n in prims():
    if n < 1000:
        print(n)
    else:
        break
```
