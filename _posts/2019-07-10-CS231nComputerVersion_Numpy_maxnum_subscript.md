---
layout: post  #这个不变
title: "CS231n计算机视觉-作业知识点（Numpy求输入数组中出现次数最多的元素及下标）" #标题
date: 2019-07-10 23:15 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

## Numpy求输入数组中出现次数最多的元素及下标
在讲之前先介绍以下几个函数：

#### numpy.argsort()

函数返回的是数组值从小到大的**索引值**。
`numpy.argsort(a, axis, kind, order)`

##### 参数：

**a：** 要排序的数组

**axis（轴）：** 沿着它排序数组的轴，如果没有数组会被展开，沿着最后的轴排序，axis=0按列排序，axis=1按行排序

**kind：** 默认为'quicksort'（快速排序）

**order：** 如果数组包含字段，则是要排序的字段

例题：
```
import numpy as np

dists = np.array([
    [0, 8, 2, 3],
    [50, 10, 6, 7]])

>>> print(np.argsort(dists))
[[0 2 1 3]
 [2 3 1 0]]
```

#### numpy.bincount()

**目的**：统计列表中元素出现的次数。

**返回结果**：返回**0-序列元素最大值**的数组中，每个元素出现的次数。

Eg: closest_y数组中含有的最大元素为9，则bincount()返回的就是0-9这10个数字出现的次数。
```
closest_y = [1, 1, 3, 3, 3, 4, 4, 8, 8, 9, 9, 9]

>>> print(np.bincount(closest_y))
[0 2 0 3 2 0 0 0 2 3]  # 0出现0次，1出现2次，2出现0次，3出现3次，以此类推，最后9出现3次
```

#### numpy.argmax和nump.argmin

numpy.argmax() 和 numpy.argmin()函数分别沿给定轴返回最大和最小元素的**索引**。
```
closest_y = [1, 1, 3, 3, 3, 4, 10, 4, 8, 8, 9, 9, 9, 10]

>>> print(np.argmax(closest_y))  #  首次出现最大值时的索引
6
>>> print(np.argmin(closest_y))  #  首次出现最小值时的索引
0
```

### 求array中出现**次数最多**的那个元素，如果有元素出现次数最多相同的情况，则选择编号较小的那个元素

Eg:其中次数最多的分别为3和9，选择较为的一个，即3，且下标为2（首次出现）
```
closest_y = [1, 1, 3, 3, 3, 4,10, 4, 8, 8, 9, 9, 9]

>>> print(closest_y)


>>> print("各个数出现的次数：", np.bincount(closest_y))  # 0-10中出现各个元素的次数
各个数出现的次数： [0 2 0 3 2 0 0 0 2 3 1]

>>> print("出现次数最多且编号较小的：", np.argmax(np.bincount(closest_y)))
出现次数最多且编号较小的： 3

>>> print("下标：",closest_y.index(np.argmax(np.bincount(closest_y))))  #元素首次出现时的下标
下标：2
```
