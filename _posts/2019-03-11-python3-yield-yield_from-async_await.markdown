---
layout: post
title:  "Python 协程到asyncio"
date:   2019-03-11 22:27:02 +0800
comments: true
tags:
- python
- asyncio
- yield
- await
- async
---

### 生成器
参考

http://note.youdao.com/noteshare?id=5d2963c2a09fe309fd47656ff6824681

http://note.youdao.com/noteshare?id=767df75916abac1ca6794c3ab8ac86dc

### 协程
协程: 这是子程序的泛化概念。协程可以在执行期间暂停，这样就可以等待外部的处理（例如IO）完成之后，从之前暂停的地方恢复执行。

### 协程的event loop
参考：https://medium.com/@lanf0n/%E5%BE%9E-asyncio-%E9%96%8B%E5%A7%8B-callback-c60a74c54743

在Asyncio模块中，每一个进程都有一个事件循环，第三方实体，管理所有的事件，在整个程序运行过程中不断循环执行，追踪事件发生的顺序将它们放到队列中，当主线程空闲的时候，调用相应的事件处理者处理事件。
比如(参见：https://python-parallel-programmning-cookbook.readthedocs.io/zh_CN/latest/chapter4/03_Event_loop_management_with_Asyncio.html)：

```
while (1) {
    events = getEvents();
    for (e in events)
        processEvent(e);
}
```

- loop = get_event_loop(): 得到当前上下文的事件循环。
- loop.call_later(time_delay, callback, argument): 延后 time_delay 秒再执行 callback 方法。
- loop.call_soon(callback, argument): 尽可能快调用 callback, call_soon() 函数结束，主线程回到事件循环之后就会马上调用 callback 。
- asyncio.set_event_loop(): 为当前上下文设置事件循环。
- asyncio.new_event_loop(): 根据此策略创建一个新的时间循环并返回。
- loop.run_forever(): 在调用 stop() 之前将一直运行。

```
import asyncio

loop = asyncio.get_event_loop()


def p(val, stop=False):
    print("valiue is:",  val)
    if stop:
        loop = asyncio.get_event_loop()
        loop.stop()

loop.call_soon(p, 'hello')
loop.call_later(2, p, 'world', True)

print(loop._ready)
print(loop._scheduled)

loop.run_forever()
```

output:

```
deque([<Handle p('hello') at /Users/yangchun/collections/src/py_grammer/py3_learning.py:6>])
[<TimerHandle when=2.210093494 p('world', True) at /Users/yangchun/collections/src/py_grammer/py3_learning.py:6>]
valiue is: hello
valiue is: world
```

deque([<Handle p('hello') at /Users/yangchun/collections/src/py_grammer/py3_learning.py:6>])和[<TimerHandle when=2.210093494 p('world', True) at /Users/yangchun/collections/src/py_grammer/py3_learning.py:6>]可以理解为队列，即todo list， 有就绪马上会被调用的的， 有还需等待一段时间才能被调用的。

### yield from 替代yield
参考

https://medium.com/@lanf0n/%E5%BE%9E-yield-%E5%88%B0-yield-from-e332684e5ba7

如下：

yield from 可以接受的 Subgenerator 也可以是任何 Iterable 的「東西」,即yield from后面必须跟iterable对象(可以是生成器，迭代器)

```
def yield_from_l(n):
  for i in range(n):
    yield i
    
g = yield_from(5) 
list(g)  # [0, 1, 2, 3, 4]
```

对比：

```
def yield_l(n):
  yield from [i for i in range(n)]
  
g = yield_l(5)  # generator
list(g)  # [0, 1, 2, 3, 4]
```

即yield from封装了单独循环可迭代对象的yield输出, yield from iterable本质上等于for item in iterable: yield item的缩写版

### asyncio.coroutine和yield from
参考

https://blog.csdn.net/soonfly/article/details/78361819

之前都是我们手工切换协程，现在当声明函数为协程后，我们通过事件循环来调度协程。

协程 = yield from + event_loop = 生成器 + 事件循环

通过@asyncio.coroutine（async）关键字定义一个协程（coroutine），协程也是一种对象。

协程不能直接运行，需要把协程加入到事件循环（loop），由后者在适当的时候调用协程。asyncio.get_event_loop方法可以创建一个事件循环，然后使用run_until_complete将协程注册到事件循环，并启动事件循环。

和asyncio.coroutine配合使用，yield from语法调用另一个协程。 
使用async或者@asyncio.coroutine修饰将普通函数包装成异步函数（协程）,但是为同步调用，便于理解；但是如果里层又要调用异步函数的话又可以用await或者yield from来调用。

python的异步调用，必须用到eventloop。

在“发出IO请求”到收到“IO完成”的这段时间里，同步IO模型下，主线程只能挂起，但异步IO模型下，主线程并没有休息，而是在消息循环中继续处理其他消息。这样，在异步IO模型下，一个线程就可以同时处理多个IO请求，并且没有切换线程的操作。

### async 和 await

```
async def dwonloader(url): 
     return "1111" 
async def download_url(url): 
    # from 
    html = await dwonloader(url) #Awaitable对象 return html 

if __name__ == "__main__": 
    coro = download_url("https://www.baidu.com") try: 
        coro.send(None) 
    except StopIteration as e: 
        result = e.value print(result)
```

```
import time,asyncio,random 

async def mygen(alist): 
    while len(alist) > 0: 
        c = randint(0, len(alist)-1) print(alist.pop(c)) 
        
a = ["aa","bb","cc"] 
c=mygen(a) 
print(c) 

输出： 
<coroutine object mygen at 0x02C6BED0>
```

但是async对生成器是无效的。async无法将一个生成器转换成协程,编程异步生成器。

```
async def mygen(alist): 
    while len(alist) > 0: 
        c = randint(0, len(alist)-1) 
        yield alist.pop(c) 
        
a = ["ss","dd","gg"] 
c=mygen(a) print(c)

输出
<async_generator object mygen at 0x02AA7170>
```

如下也为协程：

```
import time,asyncio,random \
async def mygen(alist): 
    while len(alist) > 0: 
        c = random.randint(0, len(alist)-1) print(alist.pop(c)) 
        await asyncio.sleep(1) 

strlist = ["ss","dd","gg"] 
intlist=[1,2,5,6] 
c1=mygen(strlist) 
c2=mygen(intlist) print(c1)
```
