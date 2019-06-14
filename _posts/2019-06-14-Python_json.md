---
layout: post  #这个不变
title: "Python学习——json序列化——（摘写廖雪峰老师的笔记）" #标题
date: 2019-06-14 17:24 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### Python序列化：
```
import json

# 将字典，转换为Json
d = dict(name='Bob', age=20, score=88)
print(json.dumps(d))

# 将Jsonf字符串反序列化，为字典
json_str = '{"age":20,"score":88,"name":"Bob"}'
print(json.loads(json_str))


# class类序列化
class Student(object):
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score


# 可选参数default：把任意一个对象变成一个可序列为Json的对象，因此要写一个转换函数
def student2dict(std):
    return {
        'name': std.name,
        'age': std.age,
        'score': std.score
    }


s = Student('Bob', 20, 88)
print(json.dumps(s, default=student2dict))  # dumps()中需要将转换函数传入default

# 因为通常class的实例都有一个__dict__属性，它就是dict用来存储实例变量。
# 所以这里把任意class的实例变为dict
print(json.dumps(s, default=lambda obj: obj.__dict__))


# 把Json反序列化为一个Student对象实例
def dict2student(d):
    return Student(d['name'], d['age'], d['score'])


json_str = '{"name": "Bob", "age": 20, "score": 88}'
# loads()方法首先转换出一个dict对象
# 然后，我们传入的object_hook函数负责把dict转换为Student实例
print(json.loads(json_str, object_hook=dict2student))

```
