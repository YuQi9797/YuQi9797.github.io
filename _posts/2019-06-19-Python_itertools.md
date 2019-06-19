---
layout: post  #这个不变
title: "Python学习——itertools——（摘写廖雪峰老师的笔记）" #标题
date: 2019-06-19 14:25 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### itertools模块提供了非常有用的用于操作迭代对象的函数
```
import itertools

natuals = itertools.count(1)
for n in natuals:
    print(n)
```
结果：
```
1
2
3
...
```
因为 **`count()`** 会创建一个无限的迭代器，所以上述代码会打印出自然数序列，根本停不下来，只能按Ctrl+C退出。

**`cycle()`** 会把传入的一个序列无限重复下去：
```
import itertools
cs = itertools.cycle('ABC') # 注意字符串也是序列的一种
for c in cs:
    print(c)
```
结果：
```
'A'
'B'
'C'
'A'
'B'
'C'
...
```

**`repeat()`** 负责把一个元素无限重复下去，不过如果提供第二个参数就可以限定重复次数：
```
ns = itertools.repeat('A', 3)
for n in ns:
    print(n)
```
结果：
```
A
A
A
```

无限序列虽然可以无限迭代下去，但是通常我们会通过 **`takewhile()`** 等函数根据条件判断来截取出一个有限的序列：
```
natuals = itertools.count(1)
ns = itertools.takewhile(lambda x: x <= 10, natuals)
list(ns)
```
结果：
```
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

itertools还有chain()、groupby()等迭代器操作函数。
### 作业：计算圆周率可以根据公式：
```
import itertools


def pi(N):
    ' 计算pi的值 '
    # step 1: 创建一个奇数序列: 1, 3, 5, 7, 9, ...
    odd = itertools.count(1, 2)
    # step 2: 取该序列的前N项: 1, 3, 5, 7, 9, ..., 2*N-1.
    ns = itertools.takewhile(lambda x: x <= (2 * N - 1), odd)
    # step 3: 添加正负符号并用4除: 4/1, -4/3, 4/5, -4/7, 4/9, ...
    l = list(ns)
    result = list(map(lambda x: float(4 / x * 1 if l.index(x) % 2 == 0 else 4 / x * -1), l))
    # step 4: 求和:
    return sum(result)


# 测试:
print(pi(10))
print(pi(100))
print(pi(1000))
print(pi(10000))
assert 3.04 < pi(10) < 3.05
assert 3.13 < pi(100) < 3.14
assert 3.140 < pi(1000) < 3.141
assert 3.1414 < pi(10000) < 3.1415
print('ok')

```
结果：
```
3.0418396189294032
3.1315929035585537
3.140592653839794
3.1414926535900345
ok
```
