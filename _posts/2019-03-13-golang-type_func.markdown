---
layout: post
title:  "golang 定义函数类型"
date:   2019-03-13 08:27:02 +0800
comments: true
tags:
- golang
- type
- 函数类型
---

## 定义函数类型
### 举例1

```
package main

import "fmt"

type A func(int, int)

func (f A)Serve() {
    fmt.Println("serve2")
}

func serve(int,int) {
    fmt.Println("serve1")
}

func main() {
    a := A(serve)
    a(1,2)
    a.Serve()
}
```
```
//output
serve1
serve2
```

### 举例2

```
package main

import "fmt"

type Handler interface {
	Do(k, v interface{})
}

func Each(m map[interface{}]interface{}, h Handler) {
	if m != nil && len(m) > 0 {
		for k, v := range m {
			h.Do(k, v)
		}
	}
}

type welcome string

type HandlerFunc func(k, v interface{})

func (f HandlerFunc) Do(k, v interface{}){
	f(k,v)
}

func (w welcome) selfInfo(k, v interface{}) {
	fmt.Printf("%s,我叫%s,今年%d岁\n", w,k, v)
}

func main() {
	persons := make(map[interface{}]interface{})
	persons["张三"] = 20
	persons["李四"] = 23
	persons["王五"] = 26

	var w welcome = "大家好"

	Each(persons, HandlerFunc(w.selfInfo))
	//HandlerFunc(w.selfInfo)不是方法的调用，而是转型，因为selfInfo和HandlerFunc是同一种类型，所以可以强制转型。转型后，因为HandlerFunc实现了Handler接口，所以我们就可以继续使用原来的Each方法了。
}
```

```
//output
大家好,我叫李四,今年23岁
大家好,我叫王五,今年26岁
大家好,我叫张三,今年20岁
```
