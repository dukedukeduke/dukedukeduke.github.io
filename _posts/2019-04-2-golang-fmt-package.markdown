---
layout: post
title:  "golang fmt包"
date:   2019-04-02 10:27:02 +0800
comments: true
tags:
- go
- golang
- fmt
---

#### 参考
https://www.cnblogs.com/yinzhengjie/p/7680829.html
#### 通用格式

```
%v    值的默认格式表示。当输出结构体时，扩展标志（%+v）会添加字段名
%#v   值的Go语法表示
%T    值的类型的Go语法表示
%%    百分号
```

对于％ｖ来说默认的格式是：

```
bool:                    %t
int, int8 etc.:          %d
uint, uint8 etc.:        %d, %x if printed with %#v
float32, complex64, etc: %g
string:                  %s
chan:                    %p
pointer:                 %p
```

##### 举例

```
type Sample struct {
	a   int
	str string
}
 
func main() {
	s := new(Sample)
	s.a = 1
	s.str = "hello"
	fmt.Printf("%v\n", *s)　//{1 hello}
	fmt.Printf("%+v\n", *s) //{a:1 str:hello}
	fmt.Printf("%#v\n", *s) // main.Sample{a:1, str:"hello"}
	fmt.Printf("%T\n", *s)   //main.Sample
}
```

#### 其他格式
##### 布尔值

```
%t	 true 或 false
```

##### 整数值

```
%b	二进制表示
%c	相应Unicode码点所表示的字符
%d	十进制表示
%o	八进制表示
%q	单引号围绕的字符字面值，由Go语法安全地转义
%x	十六进制表示，字母形式为小写 a-f
%X	十六进制表示，字母形式为大写 A-F
%U	Unicode格式：U+1234，等同于 "U+%04X"
```

##### 浮点数及复数

```
%b	无小数部分的，指数为二的幂的科学计数法，与 strconv.FormatFloat中的 'b' 转换格式一致。例如 -123456p-78
%e	科学计数法，例如 -1234.456e+78
%E	科学计数法，例如 -1234.456E+78
%f	有小数点而无指数，例如 123.456
%g	根据情况选择 %e 或 %f 以产生更紧凑的（无末尾的0）输出
%G	根据情况选择 %E 或 %f 以产生更紧凑的（无末尾的0）输出
字符串和bytes的slice表示：
%s	字符串或切片的无解译字节
%q	双引号围绕的字符串，由Go语法安全地转义
%x	十六进制，小写字母，每字节两个字符
%X	十六进制，大写字母，每字节两个字符
```

##### 指针

```
%p	十六进制表示，前缀 0x
```

#### 格式化函数

##### Print
Print采用默认格式将其参数格式化并写入标准输出。如果两个相邻的参数都不是字符串，会在它们的输出之间添加空格。返回写入的字节数和遇到的任何错误。

注意：Print函数并不能根据%v之类的格式符来进行格式化字符串

```
func Print(a ...interface{}) (n int, err error)
```

##### Println
Println采用默认格式将其参数格式化并写入标准输出。总是会在相邻参数的输出之间添加空格并在输出结束后添加换行符。返回写入的字节数和遇到的任何错误。

注意：Println函数并不能根据%v之类的格式符来进行格式化字符串

```
func Println(a ...interface{}) (n int, err error)
```

##### Fprint
Fprint采用默认格式将其参数格式化并写入w。如果两个相邻的参数都不是字符串，会在它们的输出之间添加空格。返回写入的字节数和遇到的任何错误。

注意：Fprint函数并不能根据%v之类的格式符来进行格式化字符串

```
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
```

#### Fprintln
Fprintln采用默认格式将其参数格式化并写入w。总是会在相邻参数的输出之间添加空格并在输出结束后添加换行符。返回写入的字节数和遇到的任何错误。

注意：Fprintln函数并不能根据%v之类的格式符来进行格式化字符串

```
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```

#### Printf
Printf根据format参数生成格式化的字符串并写入标准输出。返回写入的字节数和遇到的任何错误。

```
func Printf(format string, a ...interface{}) (n int, err error)
```

```
fmt.Printf("Hello, %v", "duke")
```

#### Fprintf
Fprintf根据format参数生成格式化的字符串并写入w。返回写入的字节数和遇到的任何错误。

```
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
```

```
fmt.Fprintf(os.Stdout, "Hello, %v", "duke")
```
#### Sprintf
Sprintf根据format参数生成格式化的字符串并返回该字符串。

```
func Sprintf(format string, a ...interface{}) string
```

```
fmt.Println(fmt.Sprintf("Hello, %v", "duke"))
```

#### Sprint
Sprint采用默认格式将其参数格式化，串联所有输出生成并返回一个字符串。如果两个相邻的参数都不是字符串，会在它们的输出之间添加空格。

```
func Sprint(a ...interface{}) string
```

#### Scanf
Scan从标准输入扫描文本，将成功读取的空白分隔的值保存进成功传递给本函数的参数。换行视为空白。返回成功扫描的条目个数和遇到的任何错误。如果读取的条目比提供的参数少，会返回一个错误报告原因。

```
func Scan(a ...interface{}) (n int, err error)
```

```
var textString string
fmt.Scan(&textString)
fmt.Println(textString)
```

#### Scanf
Scanf从标准输入扫描文本，根据format 参数指定的格式(输入必须按照指定格式，不然会报错)将成功读取的空白分隔的值保存进成功传递给本函数的参数。返回成功扫描的条目个数和遇到的任何错误。

```
func Scanf(format string, a ...interface{}) (n int, err error)
```

```
var textString string
_, err:=fmt.Scanf("Hello, %v", &textString)  //输入Hello, duke则 textstring 值为duke
fmt.Println(err)
fmt.Println(textString)
```
