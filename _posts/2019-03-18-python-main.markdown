---
layout: post
title:  "python __main__"
date:   2019-03-18 23:27:02 +0800
comments: true
tags:
- python
- main
---
### 参考

https://docs.python.org/3/tutorial/modules.html?highlight=__main__

When you run a Python module with python fibo.py <arguments>，
// 当你运行python模块用命令：python fibo.py <arguments>

the code in the module will be executed, just as if you imported it, but with the __name__ set to "__main__". 
// 模块里的代码就会被执行，就像你import了这个模块，并且将这个模块的__name__设置成__main__

### 结论
当用命令python fibo.py <arguments>执行时， 会自动将模块fibo的__name__设置成__main__
