---
layout: post
title:  "[译]golang生成随机数字，字母，slice元素"
date:   2019-04-09 14:27:02 +0800
comments: true
tags:
- golang
- random
---

#### 原文
https://yourbasic.org/golang/generate-number-random-range/
#### go随机数字基础
使用math/rand包内的rand.Seed和rand.Int63函数生成int64的非负随机数字：

```
rand.Seed(time.Now().UnixNano())
n := rand.Int63() // for example 4601851300195147788
```

类似的，rand.Float64生成随机浮点数x，范围为[0,1):

`x := rand.Float64() // for example 0.49893371771268225`

注意：如果没有使用rand.Seed做初始化，每次运行都将会得到同样的数字序列。

#### 几种不同的的随机源
math.rand包内的所有函数使用单一随机源。
如果需要，可以创建新的类型为Rand的随机生成器，并且使用其方法来产生随机数：

```
generator := rand.New(rand.NewSource(time.Now().UnixNano()))
n := generator.Int63()
x := generator.Float64()
```

#### 产生一定范围的整型和字符
##### a到b之间的数字
使用rand.Intn(m),返回随机数n，范围[0, m).

```
n := a + rand.Intn(b-a+1) // a ≤ n ≤ b
```

##### 'a'到'z'范围内的字符

`c := 'a' + rune(rand.Intn('z'-'a'+1)) // 'a' ≤ c ≤ 'z'`

#### slice的随机元素

```
chars := []rune("AB⌘123")
rand.Seed(time.Now().UnixNano())
c := chars[rand.Intn(len(chars))] 
fmt.Println(string(c))
```
