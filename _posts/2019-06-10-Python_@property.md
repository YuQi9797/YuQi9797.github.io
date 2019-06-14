---
layout: post  #这个不变
title: "Python学习——使用@property——（摘写廖雪峰老师的笔记）" #标题
date: 2019-06-10 16:52 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### 练习：
请利用@property给一个Screen对象加上width和height属性，以及一个只读属性resolution：

把一个getter方法变成属性，只需要加上@property就可以了，此时，@property本身又创建了另一个装饰器@score.setter，负责把一个setter方法变成属性赋值。
```
class Screen(object):
    @property
    def width(self):
        return self._width  # 若此处是self.width 则表示一直调用自身width方法 进入无限递归，直到溢出
        # 此处使用self._width 一是设为私有变量。二是表示这个是个属性，而非方法。

    @width.setter
    def width(self, value):
        if not isinstance(value, (int, float)):
            raise ValueError('Bad Value Type!')
        self._width = value

    @property
    def height(self):
        return self._height

    @height.setter
    def height(self, value):
        if not isinstance(value, (int, float)):
            raise ValueError('Bad Value Type!')
        self._height = value

    @property
    def resolution(self):
        return self._width * self._height


# 测试:
s = Screen()
s.width = 1024
s.height = 768
print('resolution =', s.resolution)
if s.resolution == 786432:
    print('测试通过!')
else:
    print('测试失败!')
```
