---
layout: post  #这个不变
title: "Python学习——contextlib" #标题
date: 2019-06-20 11:25 #时间
description: "Life is short,You need Python"  #说明
tag: python #这是分类标签
---

### with语句
**`with`** 语句是在Python2.6中出现的新语句。在此之前，要正确的处理涉及到异常的资源管理时，需要使用 `try...finally`:
```
try:
    f = open('/path/to/file', 'r')
    f.read()
finally:
    if f:
        f.close()
```
写 **try/finally** 过于繁琐，因此为了更简洁优美的实现资源清理，且希望这种清理工作不需要暴露给使用者，便出现了 **`with`** 语句。

with语句的基本语法结构如下：
```
with expression [as variable]:
　　　 with-block
```
所以若用with语句代替上面的try/finally的例子，可以简化为：
```
with open('/path/to/file', 'r') as f:
    f.read()
```
**`with`** 语句的expression是 **上下文管理器**，**`with`** 语句中的[as variable]是可选的，如果指定了as variable说明符，则variable是上下文管理器expression调用__enter__()函数返回的对象。所以，f并不一定就是expression，而是express.__enter__()的返回值，至于express.__enter__()返回什么就由这个函数来局决定了。with-block是执行语句，with-block执行完毕时，with语句会自动进行资源清理，对应上面例子就是with语句会自动关闭文件。

    下面我们来具体说下 **`with`** 语句在背后默默无闻地到底做了哪些事情。刚才我们说了 **expression** 是一个 **上下文管理器**，其实现了`__enter__`和`__exit__`两个函数。

    当我们调用一个 **`with`** 语句时，执行过程如下：

    1.首先生成一个上下文管理器expression，在上面例子中with语句首先以“/path/to/file”作为参数生成一个上下文管理器open("/path/to/file")。

    2.然后执行expression.__enter__()。如果指定了[as variable]说明符，将__enter__()的返回值赋给variable。上例中open("/path/to/file").__enter__()返回的是一个文件对象给f。

    3.执行with-block语句块。上例中执行读取文件。

    4.执行expression.__exit__(),在__exit__()函数中可以进行资源清理工作。上面例子中就是执行文件的关闭操作。

### 上下文管理器
上下文管理器就是实现了上下文协议的类，而上下文协议就是一个类要实现__enter__()和__exit__()两个方法。一个类只要实现了__enter__()和__exit__()，我们就称之为上下文管理器下面我们具体说下这两个方法。

　　**__enter__()**：主要执行一些环境准备工作，同时返回一资源对象。如果上下文管理器open("test.txt")的__enter__()函数返回一个文件对象。

　　**__exit__()**：完整形式为__exit__(type, value, traceback),这三个参数和调用sys.exec_info()函数返回值是一样的，分别为异常类型、异常信息和堆栈。如果执行体语句没有引发异常，则这三个参数均被设为None。否则，它们将包含上下文的异常信息。__exit_()方法返回True或False,分别指示被引发的异常有没有被处理，如果返回False，引发的异常将会被传递出上下文。如果__exit__()函数内部引发了异常，则会覆盖掉执行体的中引发的异常。处理异常时，不需要重新抛出异常，只需要返回False，with语句会检测__exit__()返回False来处理异常。

　　如果我们要自定义一个上下文管理器，只需要定义一个类并且是实现__enter__()和__exit__()即可。下面通过一个简单的例子是演示如果新建自定义的上下文管理器，我们以数据库的连接为例。在使用数据库时，有时要涉及到事务操作。数据库的事务操作当调用commit()执行sql命令时，如果在这个过程中执行失败，则需要执行rollback()回滚数据库,通常实现方式可能如下：
```
def test_write():
    con = MySQLdb.connection()
    cursor = con.cursor()
    sql = """      #具体的sql语句
    """
    try:
        cursor.execute(sql)
        cursor.execute(sql)
        cursor.execute(sql)
        con.commit()      #提交事务
    except Exception as ex:
        con.rollback()    #事务执行失败，回滚数据库
```
　　如果想通过with语句来实现数据库执行失败的回滚操作，则我们需要自定义一个数据库连接的上下文管理器，假设为DBConnection，则我们将上面例子用with语句来实现的话，应该是这样子的，如下：
```
def test_write():
    sql = """      #具体的sql语句
    """
    con = DBConnection()
    with con as cursor:   
        cursor.execute(sql)
        cursor.execute(sql)
        cursor.execute(sql)
```
　　要实现上面with语句的功能，则我们的DBConnection数据库上下文管理器则需要提供一下功能：__enter__()要返回一个连接的cursor; 当没有异常发生是，__exit__()函数commit所有的数据库操作。如果有异常发生则_exit__()会回滚数据库，调用rollback()。所以我们可以实现DBConnection如下：
```
def DBConnection(object):
    def __init__(self):
        pass

    def cursor(self):
        #返回一个游标并且启动一个事务
        pass

    def commit(self):
        #提交当前事务
        pass

    def rollback(self):
        #回滚当前事务
        pass

    def __enter__(self):
        #返回一个cursor
        cursor = self.cursor()
        return cursor

    def __exit__(self, type, value, tb):
        if tb is None:
            #没有异常则提交事务
            self.commit()
        else:
            #有异常则回滚数据库
            self.rollback()
```
### contextlib模块
## contextmanage对象
　　上文提到如果我们要实现一个自定义的上下文管理器,需要定义一个实现了__enter__和__exit__两个方法的类, 这显示不是很方便。Python的contextlib模块给我们提供了更方便的方式来实现一个自定义的上下文管理器。contextlib模块包含一个装饰器contextmanager和一些辅助函数，装饰器contextmanager只需要写一个生成器函数就可以代替自定义的上下文管理器，典型用法如下：

　　需要使用yield先定义一个生成器函数.
```
@contextmanager
def some_generator(<arguments>):
    <setup>
    try:
        yield <value>
    finally:
        <cleanup>
```
然后便可以用with语句调用contextmanage生成的上下文管理器了，with语句用法如下：
```
with some_generator(<arguments>) as <variable>:
        <body>
```
生成器函数some_generator就和我们普通的函数一样，它的原理如下：
　　1.some_generator函数在在yield之前的代码等同于上下文管理器中的__enter__函数。
　　2.yield的返回值等同于__enter__函数的返回值，即如果with语句声明了as <variable>，则yield的值会赋给variable
　　3.然后执行<cleanup>代码块，等同于上下文管理器的__exit__函数。此时发生的任何异常都会再次通过yield函数返回。

例子1：文件打开后自动管理的实现
```
@contextmanager
def myopen(filename, mode="r"):
　　  f = open(filename,mode)
　　  try:
　　      yield f
　　  finally:
　　      f.close()

with myopen("test.txt") as f:
　　  for line in f:
　　      print(line)
```
很多时候，我们希望在某段代码执行前后自动执行特定代码，也可以用@contextmanager实现。例如：
```
@contextmanager
def tag(name):
　　  print("<%s>" % name)
　　  yield
　　  print("</%s>" % name)

with tag("h1"):
　　  print("hello")
　　  print("world")
```
结果：
```
<h1>
hello
world
</h1>
```
代码的执行顺序是：

1.`with`语句首先执行`yield`之前的语句，因此打印出``<h1>``；
2.`yield`调用会执行`with`语句内部的所有语句，因此打印出`hello`和`world`；
3.最后执行`yield`之后的语句，打印出``</h1>``。

因此，`@contextmanager`让我们通过编写generator来简化上下文管理。


## closing对象
　　contextlib中还包含一个`closing`对象，这个对象就是一个上下文管理器，它的`__exit__`函数仅仅调用传入参数的`close`函数，`closing`对象的源码如下：
```
class closing(object):
    def __init__(self, thing):
        self.thing = thing
    def __enter__(self):
        return self.thing
    def __exit__(self, *exc_info):
        self.thing.close()
```
　　所以`closeing`上下文管理器仅使用于具有`close()`方法的资源对象。例如，如果我们通过`urllib.urlopen`打开一个网页，`urlopen`返回的`request`有`close`方法，所以我们就可以使用`closing`上下文管理器，如下：
```
from contextlib import closing
from urllib.request import urlopen

with closing(urlopen('https://www.python.org')) as page:
    for line in page:
        print(line)
```

`closing`也是一个经过`@contextmanager`装饰的generator，这个generator编写起来其实非常简单：
```
@contextmanager
def closing(thing):
    try:
        yield thing
    finally:
        thing.close()
```
它的作用就是把任意对象变为上下文对象，并支持`with`语句。

``@contextlib`还有一些其他decorator，便于我们编写更简洁的代码。
