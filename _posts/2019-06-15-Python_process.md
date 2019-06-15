---
layout: post  #这个不变
title: "Python学习——多进程multiprocessing——（摘写廖雪峰老师的笔记）" #标题
date: 2019-06-15 10:18 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### 多进程fork()，Only works on Unix/Linux/Mac：
```
import os

# Only works on Unix/Linux/Mac:
print('Process (%s) start...' % os.getpid())
pid = os.fork()
if pid == 0:  # 子进程用于返回0
    print('I am child process (%s) and my parent is %s.' % (os.getpid(),os.getppid()))
else:  # 父进程返回子进程ID
    print('I (%s) just created a child process (%s).' % (os.getpid(), pid))
```

### multiprocessing 跨平台的 **多进程** 支持
```
import os
from multiprocessing import Process


# 子进程要执行的代码
def run_proc(name):
    print('Run child process %s (%s)...' % (name, os.getpid()))


if __name__ == '__main__':
    print('Parent process %s.' % os.getpid())
    p = Process(target=run_proc, args=('test',))  # 创建子进程时，需要传入一个执行函数和函数的参数
    print('Child process will start.')
    p.start()  # 创建一个Process实例，用start()方法启动
    p.join()  # join()方法可以等待子进程结束后再继续往下运行，通常用于进程间的同步。
    print('Child process end.')
```
结果：
```
Parent process 8392.
Child process will start.
Run child process test (6676)...
Child process end.
```

### Pool（若要启动大量的子进程，可以用进程池来批量创建子进程）
```
from multiprocessing import Pool
import os, time, random


def long_time_task(name):
    print('Run task %s (%s)' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Task %s run %0.2f seconds' % (name, (end - start)))


if __name__ == '__main__':
    print('Parent process %s.' % os.getpid())
    p = Pool(4)  # Pool默认大小是CPU的核数，设为4，即最多同时执行4个进程。
    for i in range(5):  # 0 1 2 3 4
        p.apply_async(long_time_task, args=(i,))
    print('Waiting for all subprocesses done...')
    p.close()  # 调用join()之前必须先调用close()，调用close()之后就不能继续添加新的Process了
    p.join()  # 对Pool对象调用join()方法，会等待所有子进程执行完毕
    print('All subprocesses done.')
```
结果：
```
Parent process 12076.
Waiting for all subprocesses done...
Run task 0 (12408)
Run task 1 (10148)
Run task 2 (6456)
Run task 3 (10872)
Task 1 run 0.40 seconds
Run task 4 (10148)
Task 0 run 1.31 seconds
Task 4 run 0.94 seconds
Task 2 run 1.77 seconds
Task 3 run 2.15 seconds
All subprocesses done.
```

### 子进程
## subprocess模块，可以方便地启动一个子进程，然后控制其输入和输出
```
import subprocess

print('$ nslookup www.python.org')
r = subprocess.call(['nslookup', 'www.python.org'])
print('Exit code:', r)
```
结果：
```
$ nslookup www.python.org
��Ȩ��Ӧ��:
������:  phicomm.me
Address:  192.168.2.1

����:    dualstack.python.map.fastly.net
Address:  151.101.108.223
Aliases:  www.python.org

Exit code: 0
```

### 进程间通信
## multiprocess模块包含了底层的机制，提供了Queue、Pipes等多种方式来交换数据
## 以Queue为例，在父进程中创建两个子进程，一个往Queue里写，一个从Queue里读
```
from multiprocessing import Process, Queue
import os, time, random


# 写数据进程执行的代码：
def write(q):
    print('Process to write: %s' % os.getpid())
    for value in ['A', 'B', 'C']:
        print('Put %s to queue...' % value)
        q.put(value)
        time.sleep(random.random())


# 读数据进程执行的代码：
def read(q):
    print('Process to read: %s' % os.getpid())
    while True:
        value = q.get(True)
        print('Get %s from queue.' % value)


if __name__ == '__main__':
    # 父进程创建Queue，并传给各个子进程
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程pw，写入：
    pw.start()
    # 启动子进程pr，读取：
    pr.start()
    # 等待pw结束
    pw.join()
    # pr进程里是死循环，无法等待其结束，只能强行终止：
    pr.terminate()
```
结果：
```
Process to write: 11200
Put A to queue...
Process to read: 12608
Get A from queue.
Put B to queue...
Get B from queue.
Put C to queue...
Get C from queue.
```

### 小结：
在Unix/Linux下，可以使用fork()调用实现多进程。

要实现跨平台的多进程，可以使用multiprocessing模块。

进程间通信是通过Queue、Pipes等实现的。

在Unix/Linux下，multiprocessing模块封装了fork()调用，使我们不需要关注fork()的细节。由于Windows没有fork调用，因此，multiprocessing需要“模拟”出fork的效果，父进程所有Python对象都必须通过pickle序列化再传到子进程去，所有，如果multiprocessing在Windows下调用失败了，要先考虑是不是pickle失败了。
