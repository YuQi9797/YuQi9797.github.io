---
layout: post  #这个不变
title: "Python学习——实例属性和类属性——（摘写廖雪峰老师的笔记）" #标题
date: 2019-06-10 14:16 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### 练习：
为了统计学生人数，可以给Student类增加一个类属性，每创建一个实例，该属性自动增加：
```
class Student(object):
    count = 0

    def __init__(self, name):
        self.name = name
        Student.count += 1  #count 是类属性，实例化的是类方法中的 self. 的属性
                            #实例化时通过__init__方法下面的Student.count += 1把Student的类属性count的值改变了，所以每次实例化递增1

# 测试:
if Student.count != 0:
    print('测试失败1!')
else:
    bart = Student('Bart')
    if Student.count != 1:
        print('测试失败2!')
    else:
        lisa = Student('Bart')
        if Student.count != 2:
            print('测试失败3!')
        else:
            print('Students:', Student.count)
            print('测试通过!')

```
###小结：

实例属性属于各个实例所有，互不干扰；

类属性属于类所有，所有实例共享一个属性；

不要对实例属性和类属性使用相同的名字，否则将产生难以发现的错误。
