---
layout: post
title:  "[译]怎样使用io.Writer 接口"
date:   2019-04-05 19:27:02 +0800
comments: true
tags:
- golang
- go
- io
- writer
---

#### 原文
https://yourbasic.org/golang/io-writer-interface-explained/

#### 基础
io.Writer接口表示写入流的入口。

```
type Writer interface {
        Write(p []byte) (n int, err error)
}
```

最多写入到p的bytes的长度是len(p)， 返回实际写入的byte数和出错信息。

标准库提供了某些写入的具体实现， 也可被很多其他对象所接收。

#### 怎样使用内建writer
可以使用fmt.Fpritf函数向bytes.Buffer直接写入，
- bytes.Buffery有Write方法
- fmt.Fprintf 使用Writer对象作为第一个参数

```
var buf bytes.Buffer
fmt.Fprintf(&buf, "Size: %d MB.", 85)
s := buf.String()) // s == "Size: 85 MB."
```

类似， 可以直接写入到其他流，比如http连接， 文件等。

在go中很常见。 下一个例子， 将会利用拷贝到io.Writer的文件信息计算文件的哈希值。

```
s := "Foo"

md5 := md5.Sum([]byte(s))
sha1 := sha1.Sum([]byte(s))
sha256 := sha256.Sum256([]byte(s))

fmt.Printf("%x\n", md5)
fmt.Printf("%x\n", sha1)
fmt.Printf("%x\n", sha256)

// output

1356c67d7ad1638d816bfb822dd2c25d
201a6b3053cc1422d2c3670b62616221d2290929
1cbec737f863e4922cee63cc2ebbfaafcd1cff8b790d8cfd2e6a5d550b648afa
```

#### 优化string写入
某些标准库Writers实现有WriteString方法， 这个方法会比标准库中的Write方法更加高效， 因为它将会直接从写入string而不用分配byte的slice。

```
func WriteString(w Writer, s string) (n int, err error)
```
