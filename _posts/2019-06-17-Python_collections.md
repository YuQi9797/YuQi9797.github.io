---
layout: post  #这个不变
title: "Python学习—collections模块中的集合类—（摘写廖雪峰老师的笔记）" #标题
date: 2019-06-17 13:21 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### namedtuple
**`namedtuple`** 是一个函数，它用来创建一个自定义的 **`tuple`** 对象，并且规定了 **`tuple`** 元素的个数，并可以用属性而不是索引来引用 **`tuple`** 的某个元素。
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
我们用 **`namedtuple`** 可以很方便地定义一种数据类型，它具备tuple的不变性，又可以根据属性来引用，使用十分方便。

### deque
deque是为了高效实现插入和删除操作的双向列表，适合用于队列和栈：
```
from collections import deque

q = deque(['a', 'b', 'c'])
q.append('x')
q.appendleft('y')
print(q)
q.insert(4,'d')
print(q)
```
结果：
```
deque(['y', 'a', 'b', 'c', 'x'])
deque(['y', 'a', 'b', 'c', 'd', 'x'])
```
**`deque`** 除了实现list的 **`append()`** 和 **`pop()`** 外，还支持 **`appendleft()`** 和 **`popleft()`**，这样就可以非常高效地往头部添加或删除元素。

### defaultdict
使用 **`dict`** 时，如果引用的Key不存在，就会抛出 **`KeyError`**。如果希望key不存在时，返回一个默认值，就可以用 **`defaultdict`**：
```
from collections import defaultdict

dd = defaultdict(lambda: 'N/A')  # 注意默认值是调用函数返回的，而函数在创建defaultdict对象时传入。
dd['key1'] = 'abc'
print(dd['key1'])  # key1存在
print(dd['key2'])  # key2不存在，返回默认值
```
结果：
```
abc
N/A
```
除了在Key不存在时返回默认值，**`defaultdict`** 的其他行为跟 **`dict`** 是完全一样的。

### OrderedDict
使用 **`dict`** 时，Key是无序的。在对 **`dict`** 做迭代时，我们无法确定Key的顺序。

如果要保持Key的顺序，可以用 **`OrderedDict`**：
```
from collections import OrderedDict

d = dict([('a', 1), ('b', 2), ('c', 3)])
print(d)  # dict的Key是无序的
od = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
print(od)  # OrderedDict的Key是有序的

# 注意，OrderedDict的Key会按照插入的顺序排列，不是Key本身排序
od = OrderedDict()
od['z'] = 1
od['y'] = 2
od['x'] = 3
print(list(od.keys()))  # 按照插入的Key的顺序返回
```
结果：
```
{'a': 1, 'b': 2, 'c': 3}
OrderedDict([('a', 1), ('b', 2), ('c', 3)])
['z', 'y', 'x']
```

**`OrderedDict`** 可以实现一个FIFO的dict，当容量超出限制时，先删除最早添加的Key：
```
from collections import OrderedDict

class LastUpdatedOrderedDict(OrderedDict):

    def __init__(self, capacity):
        # 首先找到LastUpdatedOrderedDict的父类(即类OrderedDict)
        # 把类LastUpdatedOrderedDict的对象self转换为类OrderedDict的对象
        # 然后调用父类的__init__()函数————即子类调用父类的__init__()方法
        super(LastUpdatedOrderedDict, self).__init__()
        self._capacity = capacity  # 设置容量

    # 每当属性被赋值的时候都会调用该方法，
    # 因此不能再该方法内赋值 self.name = value 会死循环
    def __setitem__(self, key, value):  # 设置给定键的值
        containsKey = 1 if key in self else 0  # 若对象（该对象可看出一个dict）中有索引key，则containsKey=1
        # 只要当前容量-（1 or 0）>=固定容量，就要删除最早的键值对 （其实只要等于固定容量就要消除一个键，为了安全所以>=）
        if len(self) - containsKey >= self._capacity:  # 若新来的键值对超出了本身容量，则先按FIFO删除一个键值对
            last = self.popitem(last=False)  # last=False时，按照FIFO顺序返回
            print('remove:', last)
        # 若存在键，则删除原来的键
        if containsKey:  # 若新来的键值对未超出容量，且存在该键，则删除该键
            del self[key]
            print('set:', (key, value))
        # 若不存在键，则直接进行添加(即最后一步)
        else:  # 若未超出容量，且不存在该键，则准备添加该键
            print('add:', (key, value))
        # 最终均要调用父类的这方法，写入新来的键值对
        OrderedDict.__setitem__(self, key, value)  # 调用父类的__setitem__方法写入键值对


# __xxxitem__:使用 [''] 的方式操作属性时被调用
# __setitem__:每当属性被赋值的时候都会调用该方法，因此不能再该方法内赋值 self.name = value 会死循环
# __getitem__:当访问不存在的属性时会调用该方法
# __delitem__:当删除属性时调用该方法
ld = LastUpdatedOrderedDict(5)
ld['a'] = 'A'
ld['b'] = 'B'
ld['c'] = 'C'
ld['d'] = 'D'
ld['e'] = 'E'
ld['d'] = 'DD'
ld['g'] = 'G'
print(ld)
```
结果：
```
add: ('a', 'A')
add: ('b', 'B')
add: ('c', 'C')
add: ('d', 'D')
add: ('e', 'E')
set: ('d', 'DD')
remove: ('a', 'A')
add: ('g', 'G')
LastUpdatedOrderedDict([('b', 'B'), ('c', 'C'), ('e', 'E'), ('d', 'DD'), ('g', 'G')])

```

### ChainMap
**`ChainMap`** 可以把一组 **`dict`** 串起来并组成一个逻辑上的 **`dict`**。**`ChainMap`** 本身也是一个dict，但是查找的时候，会按照顺序在内部的dict依次查找。

什么时候使用 **`ChainMap`** 最合适？举个例子：应用程序往往都需要传入参数，参数可以通过命令行传入，可以通过环境变量传入，还可以有默认参数。我们可以用 **`ChainMap`** 实现参数的优先级查找，即先查命令行参数，如果没有传入，再查环境变量，如果没有，就使用默认参数。

下面的代码演示了如何查找user和color这两个参数：
```
from collections import ChainMap
import os, argparse

# 构造缺省参数
defaults = {
    'color':'red',
    'user':'guest'
}

# 构造命令行参数
parser = argparse.ArgumentParser()
parser.add_argument('-u', '--user')
parser.add_argument('-c', '--color')
namespace = parser.parse_args()
command_line_args = {k: v for k, v in vars(namespace).items() if v}

# 组合成ChainMap：
combined = ChainMap(command_line_args, os.environ, defaults)

# 打印参数：
print('color = %s' % combined['color'])
print('user = %s' % combined['user'])
```

### Counter
**`Counter`** 一个简单的计数器，eg：统计字符出现的个数
```
from collections import Counter
c = Counter()
for ch in 'programming':
    c[ch] = c[ch] + 1

print(c)
```

结果：
```
Counter({'r': 2, 'g': 2, 'm': 2, 'p': 1, 'o': 1, 'a': 1, 'i': 1, 'n': 1})
```
Counter实际上也是dict的一个子类，上面的结果可以看出，字符'g'、'm'、'r'各出现了两次，其他字符各出现了一次。
