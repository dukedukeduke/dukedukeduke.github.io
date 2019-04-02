---
layout: post
title:  "aliyun oss storage 创建cdn"
date:   2019-03-29 09:27:02 +0800
comments: true
tags:
- aliyun oss storage
- cdn
- bucket
---

#### 前提
1. 已经创建bucket，并且将bucket权限改为公共读
2. 需在阿里云同站上购买域名

#### cdn
进入阿里云控制台->oss对象存储管理控制台->点击相应的bucket->域名管理->绑定用户域名->输入用户域名下子域名-> enable CDN
![](https://note.youdao.com/yws/res/22532/WEBRESOURCE39e59a266fd28ca01839bbb059e2fe0a)
->提交->进入阿里云CDN控制台->查看域名管理获得cdn的域名->
![](https://note.youdao.com/yws/res/22537/WEBRESOURCEd10d22af85de0013d9eca2fd12d66508)
进入阿里云域名解析->将填入的子域名解析成cdn域名提交
![](https://note.youdao.com/yws/res/22547/WEBRESOURCE7aa150fa454dbb31df58a39eac96a677)
