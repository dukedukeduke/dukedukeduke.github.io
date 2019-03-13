---
layout: post
title:  "django 批量create"
date:   2019-03-12 22:27:02 +0800
comments: true
tags:
- python
- django
- queryset
- bulk create
---

在django1.4以后加入了新的特性。使用django.db.models.query.QuerySet.bulk_create()批量创建对象，减少SQL查询次数。改进如下：

```
querysetlist=[]
for i in resultlist:
    querysetlist.append(Account(name=i))        
Account.objects.bulk_create(querysetlist)
```
