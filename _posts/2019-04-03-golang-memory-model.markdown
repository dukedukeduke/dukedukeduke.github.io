---
layout: post
title:  "golang 内存模型（译）"
date:   2019-04-03 16:27:02 +0800
comments: true
tags:
- golang
- memory model
---

#### 原文
https://golang.org/ref/mem

#### 介绍
...
#### 建议
多个协程同时修改一个数据需要串行进行访问。
为了串行访问变量， 被访问的数据需要通过channel操作或者其他同步的方式比如sync和sync/atomic包来操作。
#### happen-before
在一个go协程内部，读取，写入都是按照程序预先指定的顺序来进行的。即，编译器和处理器在单一协程内部，当不会影响到协程程序的执行，有可能会对读写执行顺序进行重新排序。正式这种机制，在一个协程观察到额执行顺序可能与在另一个协程中获取到的顺序不一致。比如，如果在一个协程中执行了a=1;b=2;另外一个协程中b的值更新可能在a之前。
为了明确程序执行的顺序， 我们定义了“happen before”, 一个go程序在内存操作执行的部分程序执行顺序。如果事件e1比事件e2早发生， 那么我们说e2晚于e1发生，同样，如果e1不早于e2并且e1也不晚于e2发生， 我们说e1,e2同时发生。
在一个go协程中，“happen-before”顺序是被程序语言指定的。
如按下列操作，从变量v中读（事件r）,可以获得从变量v中写（事件w）入的值。
1. r不早于w
2. 没有其他写事件发生晚于w,早于r

同一协程内部没有并行，当多个协程访问同一个共享变量v时，必须使用同步机制来确保“happen before”条件。

#### 同步
##### 初始化
程序通过一个协程来进行初始化的，但是可以创建其他协程，并发工作。
如果包p导入了包q，q的init函数就会执行在p的之前。
而main函数则要等到所有的init函数执行之后才能执行。
##### 协程创建
go关键字声明一个新的协程。
比如：

```
var a string

func f() {
	print(a)
}

func hello() {
	a = "hello, world"
	go f()
}
```

调用hello会在将来某个时候打印出“hello,world”（有可能在hello函数返回之后）。
##### 协程销毁
协程退出不能保证退出时机，如下：

```
var a string

func hello() {
	go func() { a = "hello" }()
	print(a)
}
```

对a赋值不会再任何同步的事件之后，所以不能保证能被其他协程观察到。
如果一个协程的更改必须要被其他协程获取到， 用同步机制比如锁，或者channel通信来确保相对顺序。
##### channel 通信
channel是协程之间同步的主要方式， 每次发送值到特定的channel需要一次特定的取值，通常是在不同的协程中。
如下：

```
var c = make(chan int, 10)
var a string

func f() {
	a = "hello, world"
	c <- 0
}

func main() {
	go f()
	<-c
	print(a)
}
```

写入到a发生在发送到c之前，发送值到c也发生在从c中取值之前，从c取值也发生在print之前。
如果在channel关闭之后仍然取值，则会获得响应类型的零值。
前面是实例中，用close(c)代替c <- 0也会获得同样的执行结果。

```
var c = make(chan int)
var a string

func f() {
	a = "hello, world"
	<-c
}

func main() {
	go f()
	c <- 0
	print(a)
}
```

#### Locks
sync包实现了两个数据缩类型：sync.Mutex, sync.RWMutex.

```
var l sync.Mutex
var a string

func f() {
	a = "hello, world"
	l.Unlock()
}

func main() {
	l.Lock()
	go f()
	l.Lock()
	print(a)
}
```

l.Unlock()（函数f中）发生在main的第二个l.Lock()返回之前。

#### Once
sync包提供了一个Once 类型用于多协程的安全初始化机制。
对于一个特定的函数f,多个协程都可执行once.Do(f)，但是只有一个会执行，其他则会阻塞等待f()return。

```
var a string
var once sync.Once

func setup() {
	a = "hello, world"
}

func doprint() {
	once.Do(setup)
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}
```

调用了两个doprint， 但是setup只会执行一次,而“hello，world”则会打印两次。

#### 不正确的同步

```
var a, b int

func f() {
	a = 1
	b = 2
}

func g() {
	print(b)
	print(a)
}

func main() {
	go f()
	g()
}
```

有可能发生g函数打印出2和0。

```
var a string
var done bool

func setup() {
	a = "hello, world"
	done = true
}

func doprint() {
	if !done {
		once.Do(setup)
	}
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}
```

上述例子不能保证能打印出“hello，world"两次。
