---
layout: post
title:  "【译】go语言如何做面向对象"
date:   2019-04-08 11:27:02 +0800
comments: true
tags:
- golang
- go
- OO
- 面向对象
---
#### 原文
https://yourbasic.org/golang/inheritance-object-oriented/

go语言没有继承-取而代之的是组合，集成和接口设计实现重用和多态。

#### 用接口实现面向对象编程
传统的OO语言继承提供了三个特性，当Dog继承自Animal:
1. Dog类可以使用Animal类的代码
2. Animal类型的变量x能表示Dog或者Animal
3. x.Eat()会获得x类型拥有的Eat方法

在OO概念中，这些特性被称为代码继承，多态，动态调用

所有这些特性在go中都可通过构造（组合）来实现

- 构造和集成提供代码继承
- 接口则关注多态和动态调用

#### 组合实现代码继承
当开始一个新的go项目的时候， 不必担心特性层次结构-多态和动态调用都很容易理解。

如果Dog需要部分或者全部的Animal的功能， 简单构造如下：

```
type Animal struct {
	// …
}

type Dog struct {
	beast Animal
	// …
}
```

给与你在Dog中充分的使用Animal组件的空间。就这么简单。

#### 集成实现代码继承
如果Dog继承了Animal的具体行为， 可能的代码如下：

```
type Animal struct {
	// …
}

func (a *Animal) Eat()   { … }
func (a *Animal) Sleep() { … }
func (a *Animal) Breed() { … }

type Dog struct {
	beast Animal
	// …
}

func (a *Dog) Eat()   { a.beast.Eat() }
func (a *Dog) Sleep() { a.beast.Sleep() }
func (a *Dog) Breed() { a.beast.Breed() }
```

这种方式称为委托。
go集成可写成如下形式, 这样委托特性的三个方法就可以精简为如下：

```
type Dog struct {
	Animal
	// …
}
```

#### interface 实现多态和动态调用

保持interface接口的精简， 在需要的额场合再申明它。
如果你需要给你所有的宠物都加上sleep特性， 你可以定义如下Sleeper接口:

```
type Sleeper interface {
	Sleep()
}

func main() {
	pets := []Sleeper{new(Cat), new(Dog)}
	for _, x := range pets {
		x.Sleep()
	}
}
```

所有宠物提供了同一种方法， 就成为一个interface。
当我看到一只小鸟走路像一只鸭子，游泳像一只鸭子，发出嘎嘎的声音，那么我就称这只鸟是一只鸭子。
