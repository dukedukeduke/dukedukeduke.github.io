---
layout: post
title:  "google cloud storage bucket 创建cdn"
date:   2019-04-01 09:27:02 +0800
comments: true
tags:
- google storage
- cdn
- bucket
---

#### 前置条件
1. 已经申请了bucket
2. 当前用户具有CDN编辑权限
3. bucket权限需改为public,资源才能被访问

#### cdn

进入Google cloud console， 依次点击:

Network services->Cloud CDN->

ADD ORIGIN -> Select origin -> Create a load balancer -> 
![](https://note.youdao.com/yws/res/22490/WEBRESOURCEc6ccdca0715eab11cf0ee4bdac263068)
Load balancer Name ->

Backend configuration -> Create or select backend services&backend bucket -> Backend buckets -> Create a buckend bucket -> bucket Name ->Browse (select the google storage bucket) -> Enable Cloud CDN -> Create -> 
![](https://note.youdao.com/yws/res/22504/WEBRESOURCE56623a2d9c0b9ed49a0c21d58e1d82b3)
Host and path rules -> Hosts:*, Paths:/*,Bucket:[created just now] -> 
![](https://note.youdao.com/yws/res/22496/WEBRESOURCE92cd71c350e89c972a3171a361f76b4b)
Frontend configuration -> Name -> http?https(https need provide cert) -> Done -> 
![](https://note.youdao.com/yws/res/22500/WEBRESOURCE4ad10060208f64dbe3458eb29398c066)

Update

最后会获得一个ip地址， 比如：

![](http://note.youdao.com/yws/res/22482/WEBRESOURCE3d01e4aa1e09390cbcfeb28842adaad9)

#### 后续
可对该IP地址进行DNS解析
