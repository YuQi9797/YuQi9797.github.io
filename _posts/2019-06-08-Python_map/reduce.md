---
layout: post  #这个不变
title: "Python学习（三）" #标题
date: 2019-06-04 10:42 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### 利用切片操作，实现一个trim()函数，去除字符串首尾的空格，注意不要调用str的strip()方法：
```
def trim(str):
    while str[:1] == ' ':   # 从最开始取一个数，即第一个数
        str = str[1:]       # str就为从下个元素开始的字符串
    while str[-1:] == ' ':  # 取最后一个元素到结束(左向右) 即最后一个元素
        str = str[:-1]      # 从开始取数，取到最后一个字符之前（不包含最后一个字符）
    return str

# 测试:
if trim('hello  ') != 'hello':
    print('测试失败!')
elif trim('  hello') != 'hello':
    print('测试失败!')
elif trim('  hello  ') != 'hello':
    print('测试失败!')
elif trim('  hello  world  ') != 'hello  world':
    print('测试失败!')
elif trim('') != '':
    print('测试失败!')
elif trim('    ') != '':
    print('测试失败!')
else:
    print('测试成功!')
```
结果：
测试成功!

# 生成器（generator）
##第一种创建方法：()
```
g = (x * x for x in range(10))  # 把一个列表生成式的[]改为()就创建了一个generator
```

##第二种创建方法：把fib函数变成generator，只需把print(b)改为yield b
```
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b # 将print(b) 改为 yield b
        a, b = b, a + b
        n += 1
    return 'done'
```
如果一个函数定义中包含yield关键字，那么这个函数就不再是一个普通函数，而是一个generator
##练习
杨辉三角定义如下：
```
          1
         / \
        1   1
       / \ / \
      1   2   1
     / \ / \ / \
    1   3   3   1
   / \ / \ / \ / \
  1   4   6   4   1
 / \ / \ / \ / \ / \
1   5   10  10  5   1
```
把每一行看做一个list，试写一个generator，不断输出下一行的list：

```
def triangles():
    L = [1]
    while True:
        yield L
        L = [1] + [L[x] + L[x + 1] for x in range(len(L) - 1)] + [1]

n = 0
results = []
for t in triangles():
    print(t)
    results.append(t)
    n = n + 1
    if n == 10:
        break
if results == [
    [1],
    [1, 1],
    [1, 2, 1],
    [1, 3, 3, 1],
    [1, 4, 6, 4, 1],
    [1, 5, 10, 10, 5, 1],
    [1, 6, 15, 20, 15, 6, 1],
    [1, 7, 21, 35, 35, 21, 7, 1],
    [1, 8, 28, 56, 70, 56, 28, 8, 1],
    [1, 9, 36, 84, 126, 126, 84, 36, 9, 1]
]:
    print('测试通过!')
else:
    print('测试失败!')
```
