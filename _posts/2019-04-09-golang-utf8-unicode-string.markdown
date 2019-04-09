---
layout: post
title:  "如何计算中文的string长度"
date:   2019-04-09 12:27:02 +0800
comments: true
tags:
- go
- golang
- unicode
- utf8
- string
---

#### 如何计算中文的string长度

golang 默认是utf-8编码的。

golang的string的长度是按照转换为ASCII 码的数量来计算的。

比如：len("abc") == 3, 因为abc这三个字符对应的utf-8编码十进制分别为97 98 99(ascii 编码相同), 所以长度为3.

而len("你好") == 6， 这是因为'你'对应的utf-8编码十进制为14990752（十六进制为E4BDA0）， 而查询ascii码表可知对应的ascii码分别为228 189 160（E4 -> 228, BD -> 189, A0 -> 160）， 而'好'对应的utf-8编码十进制为15050173（E5A5BD），而查询ASCII码表可知对应的ASCII码分别为229 165 189。所以， "你好"长度为6， 而不是2。

那么如何计算出让结果为2呢？将string转换为Unicode，golang中， 一个rune就表示一个Unicode字符， 用单引号包围， 比如'a','你'，这就是两个Unicode字符，

`fmt.Println(reflect.TypeOf('a').String())`

所以要正确计算长度，如下：

```
var stringText string = "你好"
fmt.Println(len([]rune(stringText)))
```

#### 参考
##### utf-8编码查询
http://www.mytju.com/classCode/tools/encode_utf8.asp
##### ascii编码查询
https://www.litefeel.com/tools/ascii.php
