---
layout: post
title:  "golang 的panic和recover机制"
date:   2019-03-05 23:27:02 +0800
comments: true
tags:
- golang
- go
- panic
- recover
- 错误处理
---

从下面例子可以看出：
golang panic会自动上抛直到遇到recover, recover会获取下面一级的错误信息，并且defer中的error会自动返回到上一级

```
func funcA() (err error) {
    defer func() {
        if p := recover(); p != nil {
            fmt.Println("panic recover! p:", p)
            str, ok := p.(string)
            if ok {
                err = errors.New(str)
            } else {
                err = errors.New("panic")
            }
            debug.PrintStack()
        }
    }()
    return funcB()
}

func funcB() error {
    // simulation
    panic("foo")
    return errors.New("success")
}

func test() {
    err := funcA()
    if err == nil {
        fmt.Printf("err is nil\\n")
    } else {
        fmt.Printf("err is %v\\n", err)
    }
}

func main(){
    test()
}
```
