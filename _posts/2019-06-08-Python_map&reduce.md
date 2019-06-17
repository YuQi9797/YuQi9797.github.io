---
layout: post  #这个不变
title: "Python学习（map/reduce）——（摘写廖雪峰老师的笔记）" #标题
date: 2019-06-08 10:42 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### map()函数
map()接收两个参数，一个是函数，一个是Iterable,map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回。
```
def f(x):
    return x * x


r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
list(r)
# 结果：[1,4,9,16,25,36,49,64,81]
```
### reduce()函数
reduce()把一个函数作用在一个序列[x1,x2,x3,...]上，这个函数必须接收两个参数，reduce把结果继续和序列的下一个元素做累积计算，其效果如：
```
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)
```
如序列求和：
```
from functools import reduce
def add(x, y):
     return x + y

reduce(add, [1, 3, 5, 7, 9])
# 结果：25
```

##练习（一）
利用map()函数，把用户输入的不规范的英文名字，变为首字母大写，其他小写的规范名字。输入：['adam', 'LISA', 'barT']，输出：['Adam', 'Lisa', 'Bart']：
```
def normalize(name):
    return name[0].upper() + name[1:].lower()


L1 = ['adam', 'LISA', 'barT']
L2 = list(map(normalize, L1))  # map是将L1中的每一个参数拿出来，进入函数中计算
print(L2)
```
##练习（二）
Python提供的sum()函数可以接受一个list并求和，请编写一个prod()函数，可以接受一个list并利用reduce()求积：
```
from functools import reduce


def prod(L):
    def multiply(x, y):
        return x * y

    return reduce(multiply, L)


print('3 * 5 * 7 * 9 =', prod([3, 5, 7, 9]))
if prod([3, 5, 7, 9]) == 945:
    print('测试成功!')
else:
    print('测试失败!')
```
##练习（三）
利用map和reduce编写一个str2float函数，把字符串'123.456'转换成浮点数123.456：
```
from functools import reduce

DIGITS = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}


# 先将字符串s按照小数点划分为俩部分
# 先将s1、s2 按照整数算好
# 再通过s2的长度，将小数点标回（通过除）
def str2float(s):
    def char2num(i):
        return DIGITS[i]

    s1, s2 = s.split('.')
    num = reduce(lambda x, y: x * 10 + y, map(char2num, s1 + s2))
    return num / pow(10, len(s2))


print('str2float(\'123.456\') =', str2float('123.456'))
if abs(str2float('123.456') - 123.456) < 0.00001:
    print('测试成功!')
else:
    print('测试失败!')
```
