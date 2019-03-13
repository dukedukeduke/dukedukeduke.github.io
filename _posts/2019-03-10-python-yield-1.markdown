---
layout: post
title:  "Python 生成器"
date:   2019-03-10 18:27:02 +0800
comments: true
tags:
- python
- yield
- 生成器
---

有时候，序列或集合内的元素的个数非常巨大，如果全制造出来并放入内存，对计算机的压力是非常大的。
比如，假设需要获取一个10**20次方如此巨大的数据序列，把每一个数都生成出来，并放在一个内存的列表内，这是粗暴的方式，有如此大的内存么？
如果元素可以按照某种算法推算出来，需要就计算到哪个，就可以在循环的过程中不断推算出后续的元素，而不必创建完整的元素集合，从而节省大量的空间。
在Python中，这种一边循环一边计算出元素的机制，称为生成器：generator。
可以通过圆括号可以编写生成器推导式：

```
>>> g = (x * x for x in range(1, 4))
>>> g
<generator object <genexpr> at 0x1022ef630>
```

可以通过next()函数获得generator的下一个返回值:

```
>>> next(g)
1
>>> next(g)
4
>>> next(g)
9
>>> next(g)
Traceback (most recent call last):
  File "<pyshell#14>", line 1, in <module>
    next(g)
StopIteration
```

或者用for循环取得其值，for循环会自动调用next获取下一个值：

```
for i in g:
    print(i)
```

用yield可将普通函数变成生成器,每次遇到yield时函数会暂停并保存当前所有的运行信息，返回yield的值。并在下一次执行next()方法时从当前位置继续运行。

```
def fib_func(n):    
    a, b, counter = 0, 1, 0
    while True:
        if counter > n:
            return
        yield a             
        a, b = b, a + b
        counter += 1

fib = fib_func(10)           # fib是一个生成器
print(type(fib))
for i in fib:
    print(i, end=" ")
```


