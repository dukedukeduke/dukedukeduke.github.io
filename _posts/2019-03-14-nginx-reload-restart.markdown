---
layout: post
title:  "nginx reload与restart的区别"
date:   2019-03-14 9:27:02 +0800
comments: true
tags:
- nginx
- reload
- restart
---

### 参考
https://blog.csdn.net/ydm19891101/article/details/80589696

### 区别

使用Nginx的过程中，免不了要进行配置文件的修改，然后就是使修改的配置文件生效。
使修改的配置文件生效就需要向Nginx的master进程发送信号，具体就是reload与restart信号。

那既然两者都能使配置文件生效，又有什么区别呢？

- reload --重新加载，reload会重新加载配置文件，Nginx服务不会中断。而且reload时会测试conf语法等，如果出错会rollback用上一次正确配置文件保持正常运行。
- restart --重启（先stop后start），会重启Nginx服务。这个重启会造成服务一瞬间的中断，如果配置文件出错会导致服务启动失败，那就是更长时间的服务中断了。

---

所以，如果是线上的服务，修改的配置文件一定要备份。再者，为了保证线上服务高可用，还是使用reload吧。
