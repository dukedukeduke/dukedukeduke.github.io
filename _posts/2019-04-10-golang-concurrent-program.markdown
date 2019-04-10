---
layout: post
title:  "【译】go并发编程"
date:   2019-04-10 18:27:02 +0800
comments: true
tags:
- go
- golang
- concurrent
- 协程
---

#### 原文
https://yourbasic.org/golang/concurrent-programming/

#### 协程是轻量级线程
go关键字声明独立运行一个函数。
使用go关键字声明开始一个新的执行线程：一个协程。它会使用一个函数新创建一个协程。同一个程序中的所有协程共同使用同一个地址空间。

```go list.Sort() // Run list.Sort in parallel; don’t wait for it.```

下面的程序会打印出“Hello from main goroutine”。也可能会打印“Hello from another goroutine”， 取决于哪一个写成首先完成。

```
func main() {
    go fmt.Println("Hello from another goroutine")
    fmt.Println("Hello from main goroutine")

    // At this point the program execution stops and all
    // active goroutines are killed.
}
```

下面这个程序， 很有可能即会打印“Hello from main goroutine”也会打印“Hello from another goroutine”。可能顺序会和这里说的不一致，另外一种可能性是第二个协程运行非常慢以至于在程序运行结束之前都不能打印出它的消息。

```
func main() {
    go fmt.Println("Hello from another goroutine")
    fmt.Println("Hello from main goroutine")

    time.Sleep(time.Second) // give the other goroutine time to finish
}
```

下面是一个更接近于实际使用的例子，使用并行运行一个函数来推迟一个时间：

```
// Publish prints text to stdout after the given time has expired.
// It doesn’t block but returns right away.
func Publish(text string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println("BREAKING NEWS:", text)
    }() // Note the parentheses. We must call the anonymous function.
}
```

你可能会这样使用Publish函数：

```
func main() {
    Publish("A goroutine starts a new thread.", 5*time.Second)
    fmt.Println("Let’s hope the news will published before I leave.")

    // Wait for the news to be published.
    time.Sleep(10 * time.Second)

    fmt.Println("Ten seconds later: I’m leaving now.")
}
```

这个程序， 很可能会按如下顺序打印出三行，每一行间隔5秒：

```
$ go run publish1.go
Let’s hope the news will published before I leave.
BREAKING NEWS: A goroutine starts a new thread.
Ten seconds later: I’m leaving now.
```

实际上很难通过sleep这样的方式来组织执行的等待顺序，go语言主要通过channel来实现同步。

##### 实现
协程是轻量级的， 资源消耗仅仅比栈空间分配多一点点。一开始分配很少的栈资源， 根据需求会动态的分配和释放。go协程调度在多线程操作系统中十分复杂， 如果一个协程阻塞了一个系统线程， 例如，等待输入，当前线程中其他协程就会被调度执行。

#### channel提供通信同步

channel是go协程同步执行和通信的机制。

新的channel使用内建函数make创建：

```
// unbuffered channel of ints
ic := make(chan int)

// buffered channel with room for 10 strings
sc := make(chan string, 10)
```

使用<-操作符传递值给channel, 而从channel接收的时候，将其作为一元操作符使用。

```
ic <- 3   // Send 3 on the channel.
n := <-sc // Receive a string from the channel.
```

<-操作符指定了channel的方向，发送还是接受。如果没有指定， 则channel是双向的。

```
chan Sushi    // can be used to send and receive values of type Sushi
chan<- string // can only be used to send strings
<-chan int    // can only be used to receive ints
```

##### 有缓冲和无缓冲的channel
- 如果channel的容量是0， channel是无缓冲的并且发送会阻塞直到接收到值
- 如果channel有缓冲，阻塞会发生直到由接受者从已满的channel中取出值。
- 接受者会一直阻塞直到有数据进入channel
- 发送或者接收nil的channel会永远阻塞

##### 关闭channel
close()函数记录了没有数据会发送到channel中了，注意， 一般会是接收者来关闭channel。
- 调用close函数之后，在发送了所有接受到的数据之后，从channel接收到的是当类型的零值， 而不会阻塞
- 发送或者关闭一个已经关闭了的channel会导致运行时错误， 关闭nil channel也会导致运行时错误。

```
ch := make(chan string)
go func() {
    ch <- "Hello!"
    close(ch)
}()

fmt.Println(<-ch) // Print "Hello!".
fmt.Println(<-ch) // Print the zero value "" without blocking.
fmt.Println(<-ch) // Once again print "".
v, ok := <-ch     // v is "", ok is false.

// Receive values from ch until closed.
for v := range ch {
    fmt.Println(v) // Will not be executed.
}
```

##### 举例
下面例子中， 我们是Publish函数返回一个用于广播已经发布了的消息的channel。

```
// Publish prints text to stdout after the given time has expired.
// It closes the wait channel when the text has been published.
func Publish(text string, delay time.Duration) (wait <-chan struct{}) {
	ch := make(chan struct{})
	go func() {
		time.Sleep(delay)
		fmt.Println(text)
		close(ch)
	}()
	return ch
}
```

使用空struct的channel来指示channel将只会被用于信号，而不是传输数据，你可能会这样使用：

```
wait := Publish("important news", 2 * time.Minute)
// Do some more work.
<-wait // Block until the text has been published.
```

#### select waits on a group of channels

select会同时等待多个发送或者接收操作。
- select会阻塞直到其中一个操作被触发
- 如果多个操作被触发， 随机选择一个

```
// blocks until there's data available on ch1 or ch2
select {
case <-ch1:
    fmt.Println("Received from ch1")
case <-ch2:
    fmt.Println("Received from ch2")
}
```

发送和接收一个nil channel的操作会永远阻塞，这可用来在select中禁用某个channel。

```
ch1 = nil // disables this channel
select {
case <-ch1:
    fmt.Println("Received from ch1") // will not happen
case <-ch2:
    fmt.Println("Received from ch2")
}
```

##### 默认事件
默认事件会总是能被处理， 如果其他事件被阻塞。

```
// never blocks
select {
case x := <-ch:
    fmt.Println("Received", x)
default:
    fmt.Println("Nothing available")
}
```

##### 举例
###### 无限随机二进制序列生成
```
rand := make(chan int)
for {
    select {
    case rand <- 0: // no statement
    case rand <- 1:
    }
}
```

###### 定义阻塞超时时间
函数time.After是标准库的一部分， 它会等待一个特定的时间然后释放:

```
select {
case news := <-AFP:
    fmt.Println(news)
case <-time.After(time.Minute):
    fmt.Println("Time out: No news in one minute")
}
```

###### 永远阻塞

select会阻塞直到至少一个case能被执行， 如果没有科被执行的操作， 则会永远阻塞：

```
select {}
```

典型使用场景是在并发执行程序的main函数的最后， 当main函数退出，程序是不会等待其他协程的完成而退出的。

#### 数据竞争
当两个协程同时访问相同变量，并且至少有一个是写操作，数据竞争发生了。
数据竞争十分常见并且很难debug。

下面的函数有数据竞争， 他的行为不能被预测。有可能会打印出1。尽量指出发生了什么？

```
func race() {
    wait := make(chan struct{})
    n := 0
    go func() {
        n++ // read, increment, write
        close(wait)
    }()
    n++ // conflicting access
    <-wait
    fmt.Println(n) // Output: <unspecified>
}
```

g1, g2两个协程，发生了竞争， 并且不知道是什么样的顺序发生， 下面是多种可能的某一个输出：

```
g1	                      g2
Read the value 0 from n.	
                          Read the value 0 from n.
Incre­ment value from 0 to 1.	
Write 1 to n.	
                          Incre­ment value from 0 to 1.
                          Write 1 to n.
Print n, which is now 1.	
```

数据竞争这个名字可能会产生误解。不仅仅是操作的顺序-而是不存在确定性。编译器和硬件会调换程序的顺序以获得更好的执行效率。
##### 怎样避免数据竞争
唯一避免数据竞争的方式是同步访问数据， 可以通过几种方式来实现， 在go语言中， 可以通过channel和锁的方式（锁是更为底层的机制，在sync 和sync/atomic包内）。

go语言中更好的处理并发访问数据是使用channel来传输实际的数据到不同协程。格言是：不要通过共享内存来通信， 而要通过通信来共享内存。

```
func sharingIsCaring() {
    ch := make(chan int)
    go func() {
        n := 0 // A local variable is only visible to one goroutine.
        n++
        ch <- n // The data leaves one goroutine...
    }()
    n := <-ch // ...and arrives safely in another.
    n++
    fmt.Println(n) // Output: 2
}
```

在本例中， channel有两个作用：
- 在协程间传递数据
- 作为同步的方式

发送协程会等待其他协程接收数据， 接收携程会等待其他协程发送数据。

#### 怎样检测数据竞争

数据竞争经常遇到并且很难debug, 幸运的是， go runtime能提供帮助。

使用-race来启用内建的数据竞争检测：

```
$ go test -race [packages]
$ go run -race [packages]
```

##### 举例
下面是有数据竞争的程序：

```
package main
import "fmt"

func main() {
    i := 0
    go func() {
        i++ // write
    }()
    fmt.Println(i) // concurrent read
}
```

使用-race选项来运行这个程序， 它会告诉我们第7行和第9行发生了数据竞争：

```
$ go run -race main.go
0
==================
WARNING: DATA RACE
Write by goroutine 6:
  main.main.func1()
      /tmp/main.go:7 +0x44

Previous read by main goroutine:
  main.main()
      /tmp/main.go:9 +0x7e

Goroutine 6 (running) created at:
  main.main()
      /tmp/main.go:8 +0x70
==================
Found 1 data race(s)
exit status 66
```

##### 细节
数据竞争检测器在静态分析中不能发挥作用，它会检测实际执行代码路径的运行时的内存访问。
可以在darwin/amd64, freebsd/amd64, linux/amd64 and windows/amd64运行。

#### 死锁检测
死锁经常是在很多协程同时等待对方并且没有一个可以继续执行的情况下。

看一下下面的这种情况：

```
func main() {
	ch := make(chan int)
	ch <- 1
	fmt.Println(<-ch)
}
```

这个程序会阻塞在channel 发送操作， 永远等待某个人来读取这个值。go会通过运行时检测到这种情况， 下面是程序的输出：

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
	.../deadlock.go:7 +0x6c
```

##### debug建议
一个协程有可能被阻塞：
- 等待一个channel
- 等待锁

通常的原因如下：
- 没有其他协程能访问到channel或者锁
- 很多的协程相互等待并且没有一个能继续执行

目前go语言只能检测整个程序的冻结， 而不能检测某一部分协程被阻塞。

使用channel能比较容易的支持死锁的原因。一个程序中越多使用mutex,也就越难debug.

#### 等待协程
sync.WaitGroup等待所有的协程完成。

```
var wg sync.WaitGroup
wg.Add(2)
go func() {
    // Do work.
    wg.Done()
}()
go func() {
    // Do work.
    wg.Done()
}()
wg.Wait()
```

- main协程会调用Add来设置等待协程的数量
- 协程运行结束调用Done

同时， Wait阻塞直到这俩协程运行完成。
