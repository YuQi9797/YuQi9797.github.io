---
layout: post  #这个不变
title: "Python学习——多线程threading——（摘写廖雪峰老师的笔记）" #标题
date: 2019-06-15 15:38 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### 多线程
## Python的标准库提供了两个模块：
**_thread** 和 **threading**, 其中 **_thread** 是低级模块，**threading** 是高级模块，对 **_thread** 进行了封装。绝大多数情况下，我们只需要使用 **threading** 这个高级模块。
```
# 启动一个线程就是把一个函数传入并创建Thread实例，然后调用start()开始执行
import time, threading


# 新线程执行的代码：
def loop():
    print('thread %s is running...' % threading.current_thread().name)
    n = 0
    while n < 5:
        n = n + 1
        print('thread %s >>> %s' % (threading.current_thread().name, n))
        time.sleep(1)
    print('thread %s ended.' % threading.current_thread().name)


print('thread %s is running...' % threading.current_thread().name)
t = threading.Thread(target=loop, name='LoopThread')
t.start()
t.join()
print('thread %s ended.' % threading.current_thread().name)
```
结果：
```
thread MainThread is running...
thread LoopThread is running...
thread LoopThread >>> 1
thread LoopThread >>> 2
thread LoopThread >>> 3
thread LoopThread >>> 4
thread LoopThread >>> 5
thread LoopThread ended.
thread MainThread ended.
```
### Lock

```

```
结果：
```
Parent process 8392.
Child process will start.
Run child process test (6676)...
Child process end.
```

### Pool（若要启动大量的子进程，可以用进程池来批量创建子进程）
如果我们要确保balance计算正确，就要给change_it()上一把锁，当某个线程开始执行change_it()时，我们说，该线程因为获得了锁，因此其他线程不能同时执行change_it()，只能等待，直到锁被释放后，获得该锁以后才能改。由于锁只有一个，无论多少线程，同一时刻最多只有一个线程持有该锁，所以，不会造成修改的冲突。创建一个锁就是通过threading.Lock()来实现：
```
import time, threading

# 假定这是你的银行存款:
balance = 0
lock = threading.Lock()  # 创建一个锁


def change_it(n):
    # 先存后取，结果应该为0:
    global balance  # global声明balance是全局变量的balance.注：global语句不允许同时进行赋值eg：global balance=6
    balance = balance + n
    balance = balance - n


def run_thread(n):
    for i in range(100000):
        # 先要获取锁：
        lock.acquire()
        try:
            # 放心地改吧
            change_it(n)
        finally:
            # 改完了一定要释放锁
            lock.release()


t1 = threading.Thread(target=run_thread, args=(5,))  # 传入函数以及参数
t2 = threading.Thread(target=run_thread, args=(8,))
t1.start()
t2.start()
t1.join()
t2.join()
print('balance=', balance)
```
结果：
```
balance= 0
```
当多个线程同时执行lock.acquire()时，只有一个线程能成功地获取锁，然后继续执行代码，其他线程就继续等待直到获得锁为止。

获得锁的线程用完后一定要释放锁，否则那些苦苦等待锁的线程将永远等待下去，成为死线程。所以我们用try...finally来确保锁一定会被释放。
