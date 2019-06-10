---
layout: post  #这个不变
title: "Python学习——回文串的判断（使用filter）" #标题
date: 2019-06-08 14:02 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

###找出1-1000内的回文串：
首先将整数转换为字符串类型，最后进行分片操作
[:] 默认表示从左到右一个一个走。

[:::-1] 三个参数，其中前俩个参数表示范围从第一个数到最后一个数，第三个参数为步长，为-1，意味着是从右向左走，这点及其精妙。
```
def is_palindrome(n):
    return str(n)[:] == str(n)[::-1]


output = filter(is_palindrome, range(1, 1000))
print('1~1000:', list(output))
if list(filter(is_palindrome, range(1, 200))) == [1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 22, 33, 44, 55, 66, 77, 88, 99, 101, 111, 121, 131, 141, 151, 161, 171, 181, 191]:
    print('测试成功!')
else:
    print('测试失败!')
```
