---
layout: post  #这个不变
title: "Python学习—collections中的集合类—（摘写廖雪峰老师的笔记）" #标题
date: 2019-06-17 13:21 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### namedtuple
**`namedtuple`** 是一个函数，它用来创建一个自定义的`tuple`对象，并且规定了`tuple`元素的个数，并可以用属性而不是索引来引用`tuple`的某个元素。
```
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])  # 创建一个自定义的tuple对象
p = Point(1, 2)
print(type(p))
print(p.x)
print(p.y)
print(isinstance(p,Point))
print(isinstance(p,tuple))

# 用坐标和半径表示一个圆
# namedtuple('名称', [属性List])
Circle = namedtuple('Circle', ['x', 'y', 'r'])
```
结果：
```
<class '__main__.Point'>
1
2
True
True
```
我们用`namedtuple`可以很方便地定义一种数据类型，它具备tuple的不变性，又可以根据属性来引用，使用十分方便。
