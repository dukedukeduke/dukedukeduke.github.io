---
layout: post
title:  "Python thread local 使用"
date:   2019-03-09 19:27:02 +0800
comments: true
tags:
- python
- threading
- threading.local
---

#### refer
https://www.cnblogs.com/xybaby/p/6420873.html

在python很多的网络库中，都支持多线程，基本上都会使用到threading.local。
在python中threading.local用来表示线程相关的数据，线程相关指的是这个属性再各个线程中是独立的 互不影响，先来看一个例子:

```
class Widgt(object):
    pass

import threading
def test():
    local_data = threading.local()
    # local_data = Widgt()
    local_data.x = 1

    def thread_func():
        print('Has x in new thread: %s' % hasattr(local_data, 'x'))
        local_data.x = 2

    t = threading.Thread(target = thread_func)
    t.start()
    t.join()
    print('x in pre thread is %s' % local_data.x)

if __name__ == '__main__':
    test()
```

输出：

```
    Has x in new thread: False
    x in pre thread is 1
```

可以看到，在新的线程中 local_data 并没有x属性，并且在新线程中的赋值并不会影响到其他线程。
也可以稍微改改代码，去掉第7行的注释(`# local_data = Widgt()`)，local_data就变成了线程共享的变量。
