---
layout: post
title:  "python argparse 库"
date:   2019-04-03 14:27:02 +0800
comments: true
tags:
- python
- argparse
---

#### 设置一个解析器
使用argparse的第一步就是创建一个解析器对象，并告诉它将会有些什么参数。那么当你的程序运行时，该解析器就可以用于处理命令行参数。解析器类是 ArgumentParser 。构造方法接收几个参数来设置用于程序帮助文本的描述信息以及其他全局的行为或设置。

```
import argparse
parser = argparse.ArgumentParser(description='This is a PyMOTW sample program')
```
##### 举例
```
import argparse

parser = argparse.ArgumentParser(description='Short sample app')

parser.add_argument('-a', action="store_true", default=False)
parser.add_argument('-b', action="store", dest="b")
parser.add_argument('-c', action="store", dest="c", type=int)
parser.parse_args()

# 执行python tools.py --help , output:

usage: tools.py [-h] [-a] [-b B] [-c C]

Short sample app

optional arguments:
  -h, --help  show this help message and exit
  -a
  -b B
  -c C
```


#### 定义Positional参数
add_argument用于添加命令行参数。

add_argument常见参数：

- help:执行help命令时显示的信息
- action:表明参数运行的方式， 和可选参数同时使用， 常见"store"(参数值原样存储， 默认动作)和"store_true"（参数值存储为true/False）
1. store：保存参数值，可能会先将参数值转换成另一个数据类型。若没有显式指定动作，则默认为该动作。
2. store_const：保存一个被定义为参数规格一部分的值，而不是一个来自参数解析而来的值。这通常用于实现非布尔值的命令行标记。
3. store_ture/store_false： 保存相应的布尔值。这两个动作被用于实现布尔开关。
4. append：将值保存到一个列表中。若参数重复出现，则保存多个值。
5. append_const：将一个定义在参数规格中的值保存到一个列表中。
6. version：打印关于程序的版本信息，然后退出
- dest:参数调用的属性，如果你的参数 dest 是 "myoption"，那么你就可以args.myoption 来访问该值
- default:参数默认值
- type:定义参数的类型，比如type=int, 默认为字符串

parse_args()则返回参数的集合对象。参数类型默认为字符串，如果要指明类型，如下：

```
parser.add_argument("square", help="display a square of a given number", type=int)
```

```
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("echo"， help="echo the string you use here")
args = parser.parse_args()
print args.echo

# 执行python tools.py , output:

usage: tools.py [-h] echo
tools.py: error: too few arguments

# 执行python tools.py --help, output:

usage: prog.py [-h] echo

positional arguments:
  echo    echo the string you use here

optional arguments:
  -h, --help  show this help message and exit
  
# 执行python tools.py foo, output

foo
```

#### 定义Optional参数
可选参数未被执行设置值时，默认值为None

```
...
parser.add_argument("--verbosity", help="increase output verbosity")
...

# 调用如下：
python tools.py --verbosity 1
```
##### action
如果执行action为store_true， 那么执行--verbose则将args.verbose值设置为True, 否则为False.

```
parser.add_argument("--verbose", help="increase output verbosity",
                    action="store_true")

# 执行python tools.py --help , output:

usage: prog.py [-h] [-v]

optional arguments:
  -h, --help     show this help message and exit
  --verbose  increase output verbosity
```

##### 加上-*， 比如-v -h选项
```
parser.add_argument("-v", "--verbose", help="increase output verbosity",
                    action="store_true")
```

#### 举例
```
import argparse

parser = argparse.ArgumentParser()

# parser.add_argument('position', action='store',
#                         help='Store a position value')

parser.add_argument('-s', action='store', dest='simple_value',
        help='Store a simple value')

parser.add_argument('-c', action='store_const', dest='constant_value',
        const='value-to-store',
        help='Store a constant value')

parser.add_argument('-t', action='store_true', default=False,
        dest='boolean_switch',
        help='Set a switch to true')
parser.add_argument('-f', action='store_false', default=False,
        dest='boolean_switch',
        help='Set a switch to false')

parser.add_argument('-a', action='append', dest='collection',
        default=[],
        help='Add repeated values to a list')

parser.add_argument('-A', action='append_const', dest='const_collection',
        const='value-1-to-append',
        default=[],
        help='Add different values to list')
parser.add_argument('-B', action='append_const', dest='const_collection',
        const='value-2-to-append',
        help='Add different values to list')

parser.add_argument('--version', action='version', version='%(prog)s 1.0')

results = parser.parse_args()
# print 'position_value   =', results.position
print 'simple_value     =', results.simple_value
print 'constant_value   =', results.constant_value
print 'boolean_switch   =', results.boolean_switch
print 'collection       =', results.collection
print 'const_collection =', results.const_collection
```

output

```
$ python argparse_action.py -h

usage: argparse_action.py [-h] [-s SIMPLE_VALUE] [-c] [-t] [-f]
                          [-a COLLECTION] [-A] [-B] [--version]

optional arguments:
  -h, --help       show this help message and exit
  -s SIMPLE_VALUE  Store a simple value
  -c               Store a constant value
  -t               Set a switch to true
  -f               Set a switch to false
  -a COLLECTION    Add repeated values to a list
  -A               Add different values to list
  -B               Add different values to list
  --version        show program's version number and exit

$ python argparse_action.py -s value

simple_value     = value
constant_value   = None
boolean_switch   = False
collection       = []
const_collection = []

$ python argparse_action.py -c

simple_value     = None
constant_value   = value-to-store
boolean_switch   = False
collection       = []
const_collection = []

$ python argparse_action.py -t

simple_value     = None
constant_value   = None
boolean_switch   = True
collection       = []
const_collection = []

$ python argparse_action.py -f

simple_value     = None
constant_value   = None
boolean_switch   = False
collection       = []
const_collection = []

$ python argparse_action.py -a one -a two -a three

simple_value     = None
constant_value   = None
boolean_switch   = False
collection       = ['one', 'two', 'three']
const_collection = []

$ python argparse_action.py -B -A

simple_value     = None
constant_value   = None
boolean_switch   = False
collection       = []
const_collection = ['value-2-to-append', 'value-1-to-append']

$ python argparse_action.py --version

argparse_action.py 1.0
```
