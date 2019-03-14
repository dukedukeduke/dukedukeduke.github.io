---
layout: post
title:  "Python socket 编程"
date:   2019-02-02 18:23:02 +0800
comments: true
tags:
- python
- socket
---

#### 阻塞编程和多线程非阻塞和select实现非阻塞

改造了廖雪峰的案例:

https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386832511628f1fe2c65534a46aa86b8e654b6d3567c000

server 端：

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

'a server example which send hello to client.'

import time, socket, threading, select

WITH_THREAD = False
WITH_SELECT = True

def tcplink(sock, addr):
    print 'Accept new connection from %s:%s...' % addr
    sock.send('Welcome!')
    while True:
        data = sock.recv(1024)
        time.sleep(1)
        if data == 'exit' or not data:
            break
        sock.send('Hello, %s!' % data)
    sock.close()
    print 'Connection from %s:%s closed.' % addr

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.setblocking(True)
# 监听端口:
s.bind(('127.0.0.1', 9999))
s.listen(5)
print 'Waiting for connection...'
inputs = [s, ] # 初始化用于接受第一个请求的socket对象
while True:
    if WITH_THREAD:
        # 接受一个新连接:
        sock, addr = s.accept()
        # 创建新线程来处理TCP连接:
        t = threading.Thread(target=tcplink, args=(sock, addr))
        t.start()
    else:
        if not WITH_SELECT:
            sock, addr = s.accept()
            sock.send('Welcome!')
            while True:
                data = sock.recv(1024)
                time.sleep(1)
                if data == 'exit' or not data:
                    break
                sock.send('Hello, %s!' % data)
        else:
            s.setblocking(False)
            # r_list -- wait until ready for reading
            # w_list -- wait until ready for writing
            # x_list -- wait for an ``exceptional condition''
            r_list, w_list, e_list = select.select(inputs, [], [], 1)
            for event in r_list:
                if event == s:
                    print("新的客户端连接")
                    new_sock, addr = event.accept()
                    inputs.append(new_sock)
                else:
                    data = event.recv(1024)
                    if data:
                        event.send('Hello, %s!' % data)
                    else:
                        print("客户端断开连接")
                        inputs.remove(event)
```

client 端

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

'a socket example which send echo message to server.'

import socket, time

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 建立连接:
s.connect(('127.0.0.1', 9999))
for data in ['Michael', 'Tracy', 'Sarah']:
    # 发送数据:
    s.send(data)
    time.sleep(5)
    print s.recv(1024)
s.send('exit')
s.close()
```

启用多线程， 则会为每个连接创建一个线程， 同时处理消息； 如果不启用，则只能一个一个连接依次处理， 后一个连接必须等到前一个连接关闭之后才能响应；
而如果启用select， 则也不会依次执行，而是并发执行。

