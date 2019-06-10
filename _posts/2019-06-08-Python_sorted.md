---
layout: post  #这个不变
title: "Python学习——sorted()" #标题
date: 2019-06-08 15:02 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### sorted()函数
sorted(iterable[, cmp[, key[, reverse]]])

接收一个key函数来实现自定义的排序，例如绝对值大小排序：
```
print(sorted([36, 5, -12, 9, -21],key = abs))
# 结果 [5, 9, -12, -21, 36]
```
第三个参数reverse，为反向排序，若要进行反向排序则reverse = True
```
print(sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True))
# 结果['Zoo', 'Credit', 'bob', 'about']
```
###练习
假设我们用一组tuple表示学生名字和成绩：
L = [('Bob', 75), ('Adam', 92), ('Bart', 66), ('Lisa', 88)]

请用sorted()对上述列表分别按名字排序：
```
def by_name(t):
    return str.lower(t[0])


L = [('Bob', 75), ('Adam', 92), ('Bart', 66), ('Lisa', 88)]

L2 = sorted(L, key=by_name)
print(L2)
```

再按成绩从高到低排序：
```
def by_score(t):
    return -t[1]


L = [('Bob', 75), ('Adam', 92), ('Bart', 66), ('Lisa', 88)]

L2 = sorted(L, key=by_score)
print(L2)
```

先按成绩由高到低排序，若成绩一样，则按名字排序：
```
def by_score_name(t):
    return -t[1], t[0].lower()


L = [('Bob', 75), ('Cool', 75), ('aool', 75), ('Joey', 75), ('Adam', 92), ('Bart', 66), ('Lisa', 88)]
L2 = sorted(L, key=by_score_name)
print(L2)
```
