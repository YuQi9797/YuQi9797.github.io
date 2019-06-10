---
layout: post  #这个不变
title: "Python学习——返回函数" #标题
date: 2019-06-08 15:59 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### 利用闭包返回一个计数器函数，每次调用它返回递增整数：

理解为li = [0]只会在counterA = createCounter()赋值语句时执行，
调用counterA()只执行counter()函数，并且可以使用li这个变量。
```
def createCounter():
    num = [0]

    def counter():
        num[0] += 1
        return num[0]

    return counter


counterA = createCounter()
print(counterA(), counterA(), counterA(), counterA(), counterA())  # 1 2 3 4 5
counterB = createCounter()
if [counterB(), counterB(), counterB(), counterB()] == [1, 2, 3, 4]:
    print('测试通过!')
else:
    print('测试失败!')
```
