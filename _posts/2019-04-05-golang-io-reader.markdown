---
layout: post
title:  "[译]怎样使用io.Reader 接口"
date:   2019-04-05 19:27:02 +0800
comments: true
tags:
- go
- io
- reader
- interface
---

#### 原文
https://yourbasic.org/golang/io-reader-interface-explained/

#### 基础
io.Reader 接口是byte读取流的入口。

```
type Reader interface {
        Read(buf []byte) (n int, err error)
}
```

最多读取len(buf)字节（bytes）到buf，返回读取到的字节数，如果到流的最后，会返回一个io.EOF的error。

go标准库提供了Reader的实现（包括内存缓冲，文件，网络连接），并且Reader可被多种方式接受（HTTP client 和server实现）。

#### 使用内建的reader

使用strings.Reader函数可以创建一个从string读取的Reader, 可将这个Reader传给net/http的http.Post函数， 而Reader就可被用作被发送数据的源。

```
r := strings.NewReader("my request")
resp, err := http.Post("http://foo.bar",
	"application/x-www-form-urlencoded", r)
```

http.Post使用Reader对象代替了[]byte， 当然也可用文件内容来代替。

#### 从字节流读取

使用Read函数直接读取

```
r := strings.NewReader("abcde")

buf := make([]byte, 4)
for {
	n, err := r.Read(buf)
	fmt.Println(n, err, buf[:n])
	if err == io.EOF {
		break
	}
}

// output

4 <nil> [97 98 99 100]
1 <nil> [101]
0 EOF []
```

使用io.ReadFull来去取特定len(buf)个字节到buf.

```
r := strings.NewReader("abcde")

buf := make([]byte, 4)
if _, err := io.ReadFull(r, buf); err != nil {
	log.Fatal(err)
}
fmt.Println(buf)

if _, err := io.ReadFull(r, buf); err != nil {
	fmt.Println(err)
}

//output

[97 98 99 100]
unexpected EOF
```

使用ioutil.ReadAll来读取一切：

```
r := strings.NewReader("abcde")

buf, err := ioutil.ReadAll(r)
if err != nil {
	log.Fatal(err)
}
fmt.Println(buf)

// output

[97 98 99 100 101]
```

#### 带缓存的读和扫描
bufio.Reader和bufio.Scanner类型包装了Reader对象， 实现了另外一个具有缓存和帮助信息的Reader,在这个例子中， 使用bufio.Scanner来计数文本中具有的word数量：

```
const input = `Beware of bugs in the above code;
I have only proved it correct, not tried it.`

scanner := bufio.NewScanner(strings.NewReader(input))
scanner.Split(bufio.ScanWords) // Set up the split function.

count := 0
for scanner.Scan() {
    count++
}
if err := scanner.Err(); err != nil {
    fmt.Println(err)
}
fmt.Println(count)

//output

16
```
