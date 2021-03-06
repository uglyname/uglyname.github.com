---
title: Python Generator(生成器) 
layout: post
category: 编程语言
tags: Python
---

###生成器是什么



生成器是一个这样的函数：它包含yield表达式，可以随着时间**返回一系列值**。


与普通的函数所不同的是：普通函数包含return语句，返回一个值之后就结束函数，而生成器返回一个值后不会结束函数（直达引发异常或碰到return语句），生成器yield一个值之后，向它的调用者返回这个值，函数自动挂起，然后保存足够的状态信息（例如：本地变量的值）使得函数下次可以从此处恢复继续运行。


在Python中，生成器被自动用作实现迭代协议，因此它只能够在迭代语境中出现。



###实例



看下面一段代码

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

def myGenerator():
    for i in range(3):
        yield i

for i in myGenerator():
    print i

#0
#1
#2
```

直观的解释就是：生成器函数在执行的时候，碰到yield时会生成一个值，并返回给调用者，同时函数在此处停住并且保存当前的运行状态，等到下一次再运行的时候，直接从此处开始运行，直到下一次碰到yield语句。



如果想要看看for循环里面发生了什，在终端里面直接调用生成器看看

```python
>>> def myGenerator():
        print "foo"
...     for i in range(3):
...             yield i
... 
>>> myGenerator()
<generator object myGenerator at 0xb741a0cc>
>>> 
```

并没有执行函数。得到的是个生成器对象，它支持迭代协议，也就是说它具有next()方法，可以对它进行迭代直到产生异常

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

def myGenerator():
    for i in range(3):
        yield i

f = myGenerator()

print f.next()
print f.next()
print f.next()
print f.next()

# 0
# 1
# 2
# Traceback (most recent call last):
#   File "1.py", line 13, in <module>
#     print f.next()
# StopIteration
```


###生成器工作机制


使用for循环来遍历一个对象发生以下事情:

>1.for循环通过调用对象的\_\_iter\_\_()方法来获得一个可迭代对象，如果返回值为None说明该对象不支持迭代协议。

>2.如果对象支持迭代协议，那么for循环自动反复调用\_\_iter()\_\_返回的对象的next()方法，直到产生异常时循环结束。

>3.如果对象不支持迭代协议，那么for循环就会使用索引协议对其进行迭代。



生成器支持迭代协议，所以用for对生成器进行迭代时，自动反复调用next()方法，这就是为什么for和生成器在一起可以工作的原因。



清楚这一点之后，我们完全利用它自己实现一个可迭代对象，用for遍历，原则就是：**这个对象要有next()方法供for循环调用**。

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

class myClass:

    def __iter__(self):
        self.i = 0
        return self

    def next(self):
        if self.i > 3:
            raise StopIteration
        self.i += 1
        return self.i


T = myClass()

for i in T:
    print i

# 1
# 2
# 3
# 4
```



至此也就明白了为什么前面直接调用生成器函数没有运行的原因，因为生成器是通过next()方法来工作的，换句话说就是必须调用next()来开启函数。



**可以看出，生成器是一个可迭代对象。实现了迭代协议，具有next()方法，使用它我们可以获得一系列值，它可以省去我们手动定义可迭代类的麻烦**。



需要注意的是：我们只能对生成器进行一次遍历，因为它的执行是通过调用next()方法来工作的，所以遍历一次后生成器的状态就会发生变化(例如已经迭代到了对象的末尾)，当然不能再继续遍历。生成器yield的值是动态生成的，而不是一次性加载到内存，所以要遍历很大(可能无法一次装入内存)的对象时可以使用生成器，相比列表它更节省内存。



###生成器表达式


从语法的角度来讲，生成器表达式和一般的列表解析一样，但是它是在圆括号而不是方括号中。

>列表解析

```python

>>> [x ** 2 for x in range(3)]
[0, 1, 4]
```

>生成器表达式

```python
>>> (x ** 2 for x in range(3))
<generator object <genexpr> at 0xb748207c>
```

尽管这样，从执行的过程来讲很不相同：生成器表达式不是在内存中构建结果，而是返回一个生成器对象，这个对象会支持迭代协议并在任意迭代语境的操作中，获得最终结果列表中的一部分。

