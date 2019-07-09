---
layout: post  #这个不变
title: "CS231n计算机视觉-作业知识点（一）" #标题
date: 2019-07-09 21:43 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

## Numpy的常用函数reshape
```
numpy.reshape(a, newshape, order='C')
```
Gives a new shape to an array without changing its data.

##### 参数：

**a：** 需要重塑的张量

**newshape：** int or tuple of ints   (即int或者int的元组)。

数组新的shape属性应该要与原来的配套，如果等于-1的话，那么Numpy会根据剩下的维度计算出数组的另外一个shape属性值

**order：**{'C', 'F', 'A'}, optional   (即可选项，暂时不说这个)

#### 例子：
有一个数组z，它的shape属性为(4, 4)
```
>>> z = np.array([
  [1, 2, 3, 4],
  [5, 6, 7, 8],
  [9, 10, 11, 12],
  [13, 14, 15, 16]])
>>> z.shape
(4,4)
```

**z.reshape(-1)**:
```
>>> z.reshape(-1)
array([ 1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14, 15, 16])
```

**z.reshape(-1,2)**:
先前我们不知道z的shape属性是多少，但是想让z变成只有2列，行数不知道是多少，通过
z.reshape(-1,2)，Numpy自动计算出有8行，新的数组shape属性为(8,2)，与原来的(4,4)配套。
```
>>> z.reshape(-1,2) #我们不知道z的shape属性是多少，
                    #但是想让z变成只有2列，行数不知道多少，
                    #通过`z.reshape(-1,2)`，Numpy自动计算出有8行，
                    #新的数组shape属性为(8, 2)，与原来的(4, 4)配套。
array([[ 1,  2],
       [ 3,  4],
       [ 5,  6],
       [ 7,  8],
       [ 9, 10],
       [11, 12],
       [13, 14],
       [15, 16]])
```
#### CS231n计算机视觉homework1中：
```
num_training = 5000
mask = list(range(num_training))
X_train = X_train[mask]
y_train = y_train[mask]

num_test = 500
mask = list(range(num_test))
X_test = X_test[mask]
y_test = y_test[mask]

# Reshape the image data into rows
X_train = np.reshape(X_train, (X_train.shape[0], -1))
X_test = np.reshape(X_test, (X_test.shape[0], -1))
print(X_train.shape, X_test.shape)
```
5000

(5000,3072)  (500,3072)

其中：
```
X_train = np.reshape(X_train, (X_train.shape[0], -1))
```
即将**X_train**进行张量变形，形状为**(X_train.shape[0], -1)**即(5000, -1)。
也就是5000行，列不知道，根据剩下纬度进行计算，即**32x32x3=3072**。所以最终形状为 **(5000,3072)**。
