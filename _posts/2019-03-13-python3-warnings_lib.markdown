---
layout: post
title:  "Python3 warnings库"
date:   2019-03-13 8:27:02 +0800
comments: true
tags:
- python3
- lib
- warnings
---

```
class WarningL(object):
    def __init__(self):
        warnings.warn("@WarningL is deprecated, use xxx instead",
                      DeprecationWarning,
                      stacklevel=2)


a = WarningL()   # 调用WarningL就会出现WarningL is deprecated, use xxx instead 的警告字样
```
