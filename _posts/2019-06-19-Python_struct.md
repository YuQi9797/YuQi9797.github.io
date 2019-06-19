---
layout: post  #这个不变
title: "Python学习——struct——（摘写廖雪峰老师的笔记）" #标题
date: 2019-06-19 12:25 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### struct模块
**`struct`** 模块来解决bytes和其他二进制数据类型的转换。

**`struct`** 的 **`pack`** 函数把任意数据类型变成bytes：

'>I'意思是：>表示字节顺序是big-endian(大端)，也就是网络序，I表示4字节无符号整数。
```
import struct
print(struct.pack('>I',1024009))  # '>I'意思是：>表示字节顺序是big-endian(大端)，也就是网络序，I表示4字节无符号整数。
```
结果：
```
b'\x00\x0f\xa0\t'
```

**`struct`** 的 **`upack`** 把bytes变成相应的数据类型:

根据>IH的说明，后面的bytes依次变为I：4字节无符号整数和H：2字节无符号整数。
```
import struct
print(struct.unpack('>IH',b'\xf0\xf0\xf0\xf0\x80\x80'))  # 根据>IH的说明，后面的bytes依次变为I：4字节无符号整数和H：2字节无符号整数。
```
结果：
```
(4042322160, 32896)
```
### 作业：打印图片大小和颜色数
首先找一个bmp文件，读入前30个字节来分析：`s = b'\x42\x4d\x38\x8c\x0a\x00\x00\x00\x00\x00\x36\x00\x00\x00\x28\x00\x00\x00\x80\x02\x00\x00\x68\x01\x00\x00\x01\x00\x18\x00'`
BMP格式采用小端方式存储数据，文件头的结构按顺序如下：
两个字节：**'BM'** 表示Windows位图，**'BA'** 表示OS/2位图； 一个4字节整数：表示位图大小； 一个4字节整数：保留位，始终为0； 一个4字节整数：实际图像的偏移量； 一个4字节整数：Header的字节数； 一个4字节整数：图像宽度； 一个4字节整数：图像高度； 一个2字节整数：始终为1； 一个2字节整数：颜色数。

所以，组合起来用unpack读取：
### 例题：
```
s = b'\x42\x4d\x38\x8c\x0a\x00\x00\x00\x00\x00\x36\x00\x00\x00\x28\x00\x00\x00\x80\x02\x00\x00\x68\x01\x00\x00\x01\x00\x18\x00'

struct.unpack('<ccIIIIIIHH', s)
```
结果：
```
(b'B', b'M', 691256, 0, 54, 40, 640, 360, 1, 24)
```
结果显示，b'B'、b'M'说明是Windows位图，位图大小为640x360，颜色数为24。

### 请编写一个test.py,可以检查任意文件是否是位图文件，如果是，打印出图片大小和颜色数。
```
import base64, struct

bmp_data = base64.b64decode(
    'Qk1oAgAAAAAAADYAAAAoAAAAHAAAAAoAAAABABAAAAAAADICAAASCwAAEgsAAAAAAAAAAAAA/3//f/9//3//f/9//3//f/9//3//f/9//3//f/9//3//f/9//3//f/9//3//f/9//3//f/9//3//f/9/AHwAfAB8AHwAfAB8AHwAfP9//3//fwB8AHwAfAB8/3//f/9/AHwAfAB8AHz/f/9//3//f/9//38AfAB8AHwAfAB8AHwAfAB8AHz/f/9//38AfAB8/3//f/9//3//fwB8AHz/f/9//3//f/9//3//f/9/AHwAfP9//3//f/9/AHwAfP9//3//fwB8AHz/f/9//3//f/9/AHwAfP9//3//f/9//3//f/9//38AfAB8AHwAfAB8AHwAfP9//3//f/9/AHwAfP9//3//f/9//38AfAB8/3//f/9//3//f/9//3//fwB8AHwAfAB8AHwAfAB8/3//f/9//38AfAB8/3//f/9//3//fwB8AHz/f/9//3//f/9//3//f/9/AHwAfP9//3//f/9/AHwAfP9//3//fwB8AHz/f/9/AHz/f/9/AHwAfP9//38AfP9//3//f/9/AHwAfAB8AHwAfAB8AHwAfAB8/3//f/9/AHwAfP9//38AfAB8AHwAfAB8AHwAfAB8/3//f/9//38AfAB8AHwAfAB8AHwAfAB8/3//f/9/AHwAfAB8AHz/fwB8AHwAfAB8AHwAfAB8AHz/f/9//3//f/9//3//f/9//3//f/9//3//f/9//3//f/9//3//f/9//3//f/9//3//f/9//3//f/9//38AAA==')


def bmp_info(data):
    # 读入前30个字节来分析
    data = data[:30]
    t = struct.unpack('<ccIIIIIIHH', data)
    if t[0] == b'B' and t[1] ==b'M':
        return {
            'width': t[6],
            'height': t[7],
            'color': t[9]
        }
    else:
        print('It is not BMP.')
        return


# 测试
bi = bmp_info(bmp_data)
assert bi['width'] == 28
assert bi['height'] == 10
assert bi['color'] == 16
print('ok')
```
