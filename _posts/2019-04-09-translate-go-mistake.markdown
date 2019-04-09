---
layout: post
title:  "【译】你使用go语言可能正在犯的错"
date:   2019-04-09 10:27:02 +0800
comments: true
tags:
- go
- golang
- mistake
---

#### 原文
https://yourbasic.org/golang/gotcha/

#### nil map进行赋值
为什么如下程序会panic?

```
var m map[string]float64
m["pi"] = 3.1416

// output

panic: assignment to entry in nil map
```

##### 答案

在使用map添加元素之前，必须用make函数进行初始化。

```
m := make(map[string]float64)
m["pi"] = 3.1416
```

一个新申明空的map,需要使用内建函数make进行初始化，该函数接收map类型和一个可选的容量作为参数：

```
make(map[string]int)
make(map[string]int, 100)
```

初始容量并不会固定map的元素数量，maps会随着元素数量的增加而增长。nil map相当于一个只读的空map。

#### Invalid memory address or nil pointer dereference
为什么如下程序会拋错？

```
type Point struct {
    X, Y float64
}

func (p *Point) Abs() float64 {
    return math.Sqrt(p.X*p.X + p.Y*p.Y)
}

func main() {
    var p *Point
    fmt.Println(p.Abs())
}

//output

panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0xffffffff addr=0x0 pc=0xd2c5a]

goroutine 1 [running]:
main.(*Point).Abs(...)
	../main.go:6
main.main()
	../main.go:11 +0x1a
```

##### 答案
man函数中未初始化的指针p是nil， 你不能对该指针取值。

如果x是nil, 尝试通过*x取值会获得一个运行时的panic,参考：https://golang.org/ref/spec#Address_operators

需要创建Point:

```
func main() {
    var p *Point = new(Point)
    fmt.Println(p.Abs())
}
```

因为指针作为方法的接收者， 方法可被值或者指针被调用， 所以你也可以用如下方式：

```
func main() {
    var p Point // has zero value Point{X:0, Y:0}
    fmt.Println(p.Abs())
}
```

#### Multiple-value in single-value context
为什么如下代码会编译错误？

```
t := time.Parse(time.RFC3339, "2018-04-06T10:49:05Z")
fmt.Println(t)

//output

../main.go:9:17: multiple-value time.Parse() in single-value context
```

##### 答案

time.Parse()函数返回两个值， time.Time类型值和error值， 你必须对两者都进行处理：

```
t, err := time.Parse(time.RFC3339, "2018-04-06T10:49:05Z")
if err != nil {
    // TODO: Handle error.
}
fmt.Println(t)

//output

2018-04-06 10:49:05 +0000 UTC
```

##### 忽略返回值
你可使用空白标识符来忽略不需要的返回值。

```
m := map[string]float64{"pi": 3.1416}
_, exists := m["pi"] // exists == true
```

#### 数组值未修改

```
func Foo(a [2]int) {
    a[0] = 8
}

func main() {
    a := [2]int{1, 2}
    Foo(a)         // Try to change a[0].
    fmt.Println(a) // Output: [1 2]
}
```

##### 答案
- go语言中数组是值类型
- 当你需要传数组到一个函数， 数组会被拷贝
如果需要使Foo更新变量a，使用slice来进行传值。

```
func Foo(a []int) {
    if len(a) > 0 {
        a[0] = 8
    }
}

func main() {
    a := []int{1, 2}
    Foo(a)         // Change a[0].
    fmt.Println(a) // Output: [8 2]
}
```

slice 不会存取任何的数据， 它只是对数组进行了区间记录。
当改变一个slice的元素，就会修改响应数组的元素值，同时引用这个数组的其他slice的值也会感知到这个改变。

更多slice相关请参考https://yourbasic.org/golang/slices-explained/。

#### Shadowed variables
为什么n值不会被修改？

```
func main() {
    n := 0
    if true {
        n := 1
        n++
    }
    fmt.Println(n) // 0
}
```

if中初始化语句n := 1声明了一个新的变量，覆盖了外层变量n。
为了使用外层变量n， 用n = 1 代替：

```
func main() {
    n := 0
    if true {
        n = 1
        n++
    }
    fmt.Println(n) // 2
}
```

##### 检测被覆盖的变量
为了帮助检测被覆盖的变量， 可以使用vet 工具提供的experimental -shadow特性
。它会标记可能被无意覆盖的变量， 前面的原始代码通过vet检测给出了如下告警信息：

```
$ go vet -shadow main.go
main.go:4: declaration of "n" shadows declaration at main.go:2
```

go1.12不再支持， 但是你可以用如下库进行：

```
go install golang.org/x/tools/go/analysis/passes/shadow/cmd/shadow
go vet -vettool=$(which shadow)
```

另外，go编译器也会检测并且不允许某些覆盖的情况， 如下:

```
func Foo() (n int, err error) {
    if true {
        err := fmt.Errorf("Invalid")
        return
    }
    return
}

//output

../main.go:4:3: err is shadowed during return
```

#### Unexpected newline
为什么如下程序会编译错误？

```
func main() {
    fruit := []string{
        "apple",
        "banana",
        "cherry"
    }
    fmt.Println(fruit)
}

//output

../main.go:5:11: syntax error: unexpected newline, expecting comma or }
```

##### 答案

在多行的slice， 数组， 或者map，每一行必须以逗号结尾。

```
func main() {
    fruit := []string{
        "apple",
        "banana",
        "cherry", // comma added
    }
    fmt.Println(fruit) // "[apple banana cherry]"
}
```

这个规则是semicolon insertion rules进行约定的，https://golang.org/ref/spec#Semicolons， 这样的话，当要删除某一行数据，不需要去其他行进行修改。

#### Immutable strings
为什么如下程序编译错误？

```
s := "hello"
s[0] = 'H'
fmt.Println(s)

//output

../main.go:3:7: cannot assign to s[0]
```

##### 答案
go字符串是类似只读的bytes 切片（还额外具有一些其他特征）。如果要更新这个数据， 用rune类型的slice代替：

```
buf := []rune("hello")
buf[0] = 'H'
s := string(buf)
fmt.Println(s)  // "Hello"
```

如果字符串仅仅是包含ASCII码吗，也可使用byte类型的slice。

#### How does characters add up?
为什么如下打印语句输出不一样？

```
fmt.Println("H" + "i")
fmt.Println('H' + 'i')

// Output

Hi
177
```

##### 答案
rune类型的'H','i'是Unicode编码的整型值， 'H' 是72， 'i'是105.
可以对其进行强制转化：

```
fmt.Println(string(72) + string('i')) // "Hi"
```

也可以用fmt.spritf函数实现：

```
s := fmt.Sprintf("%c%c, world!", 72, 'i')
fmt.Println(s)// "Hi, world!"
```

更多fmt标志和格式化方法参考：https://yourbasic.org/golang/fmt-printf-reference-cheat-sheet/

#### What happened to ABBA?
string.TrimRight 发生了什么？

```
fmt.Println(strings.TrimRight("ABBA", "BA")) // Output: ""
```

##### 答案

Trim,TrimLeft，TrimRight函数会丢弃所有的cutset中的Unicode字符码点， 在本例中， A:s和B:s都会从string中丢弃，最后得到一个空字符串。
如果需要去除尾部的string,使用strings.TrimSuffix：

```
fmt.Println(strings.TrimSuffix("ABBA", "BA")) // Output: "AB"
```

更多strings 函数参考: https://yourbasic.org/golang/string-functions-reference-cheat-sheet/

#### Where is my copy?
为什么copy结果消失了？

```
var src, dst []int
src = []int{1, 2, 3}
copy(dst, src) // Copy elements to dst from src.
fmt.Println("dst:", dst)

//output

dst: []
```

被拷贝的元素数量将是dst和src长度的最小值， 如果要实现一个完整拷贝，必须分配一个足够大的目标slice:

```
var src, dst []int
src = []int{1, 2, 3}
dst = make([]int, len(src))
n := copy(dst, src)
fmt.Println("dst:", dst, "(copied", n, "numbers)")

//output

dst: [1 2 3] (copied 3 numbers)
```

copy函数的返回值是被拷贝的元素的数量，参考copy内建方法：https://yourbasic.org/golang/copy-explained/。

##### 使用append
可以使用append函数实现复制到一个nil 的slice：

```
var src, dst []int
src = []int{1, 2, 3}
dst = append(dst, src...)
fmt.Println("dst:", dst)

//output

dst: [1 2 3]
```

#### Why doesn’t append work every time?
append函数发生了什么事？

```
a := []byte("ba")

a1 := append(a, 'd')
a2 := append(a, 'g')

fmt.Println(string(a1)) // bag
fmt.Println(string(a2)) // bag
```

如果有足够的空间， append会沿用已绑定的数组地址，如下：

```
a := []byte("ba")
fmt.Println(len(a), cap(a)) // 2 32
```

这就表示上例中a,a1,a2都会指向同一个底层数组。

为了避免这个，可以用两个独立的byte 数组：

```
const prefix = "ba"

a1 := append([]byte(prefix), 'd')
a2 := append([]byte(prefix), 'g')

fmt.Println(string(a1)) // bad
fmt.Println(string(a2)) // bag
```

#### Constant overflows int
为什么如下代码不能编译？

```
const n = 9876543210 * 9876543210
fmt.Println(n)

//output

../main.go:2:13: constant 97546105778997104100 overflows int
```

未指定类型的常量n在赋值给interface{}参数必须转换成特定类型：

```
fmt.Println(a ...interface{})
```

当类型没有参考的情况下， 未指定的常量类型会被指定为bool, int, float64, complex128, string 或者rune， 具体取决于格式方式。

在这里， 常量是一个整型， 但是n大于了int最大值。
然而，n可以用float64()进行转换：

```
const n = 9876543210 * 9876543210
fmt.Println(float64(n))

//output:

9.75461057789971e+19
```

对于特定的类型的最大值， math/big(https://golang.org/pkg/math/big/)中有所定义。

#### Unexpected ++, expecting expression
如下代码为什么不能编译？

```
i := 0
fmt.Println(++i)
fmt.Println(i++)

//output

main.go:9:14: syntax error: unexpected ++, expecting expression
main.go:10:15: syntax error: unexpected ++, expecting comma or )
```

##### 答案
go语言的自增和自减操作符不能被用作表达式， 只能用作声明。
可以修改为如下：

```
i := 0
i++
fmt.Println(i)
fmt.Println(i)
i++
```

#### Get your priorities right
为什么如下计算小时䄦秒的程序不能运行？

```
n := 43210 // time in seconds
fmt.Println(n/60*60, "hours and", n%60*60, "seconds")

//output

43200 hours and 600 seconds
```

##### 答案
*，/,%操作符具有相同的优先级， 并且从左到右进行计算， n/60*60相当于（n/60）*60.

插入括号强制确定计算顺序。
```
fmt.Println(n/(60*60), "hours and", n%(60*60), "seconds")
//output
12 hours and 10 seconds
```

或者更好的方法：

```
const SecPerHour = 60 * 60
fmt.Println(n/SecPerHour, "hours and", n%SecPerHour, "seconds")

//output

12 hours and 10 seconds
```

#### Go and Pythagoras
毕达哥拉斯定理： 
```math
a2+ b2 = c2
```

```
fmt.Println(3^2+4^2 == 5^2) // true
```

但是当改为（6，8，10），go貌似不赞同这个定理了：

```
fmt.Println(6^2+8^2 == 10^2) // false
```
 
 ##### 答案
 ^ 在go语言中表示XOR：
 
```
0011 ^ 0010 == 0001   (3^2 == 1)
0100 ^ 0010 == 0110   (4^2 == 6)
0101 ^ 0010 == 0111   (5^2 == 7)
```

当然 1 +6 ==7。
为了实现power2，可以如下：

```
fmt.Println(6*6 + 8*8 == 10*10) // true
```
 
go没有内建函数支持power（次方）的计算， 但是math包的math.Pow函数可以实现。
 
#### No end in sight
为什么下列语句会无线循环？
 
 ```
 var b byte
for b = 250; b <= 255; b++ {
    fmt.Printf("%d %c\n", b, b)
}
 ```
 
##### 答案
当b ==255的时候， b++执行， 会导致溢出（byte最大值是255）使得b ==0, 这样b<=255就会一直成立。如果将b<=255改成b<256， 编译器会报错（不同IDE可能结果不一样，但是都不能正常执行， 因为两个类型不能进行比较，256已经超过了byte的最大范围）
 
#### Numbers that start with zero
如下计数程序会发生什么？
 
```
const (
    Century = 100
    Decade  = 010
    Year    = 001
)
// The world's oldest person, Emma Morano, lived for a century,
// two decades and two years.
fmt.Println("She was", Century+2*Decade+2*Year, "years old.")

//output

She was 118 years old
```
 
010是八进制的8，而不是十进制的10.
go语言中整形可以有八进制，十进制，十六进制，比如数字16可以表示为020， 16和0x10.
八进制以0开头，十六进制以0x开头。
 
#### Whatever remains
为什么-1不是奇数？
 
```
 func Odd(n int) bool {
    return n%2 == 1
}

func main() {
    fmt.Println(Odd(-1)) // false
}
```
 
如果除法除数(下面的x)是负数， 求余操作符结果也是负数：如果n是奇负数， n%2 == -1.
 
 q = x/y, r = x%y,则
 
 ```
 x = q*y + r  and  |r| < |y|
 ```
 
 ```
 x     y     x / y     x % y
 5     3       1         2
-5     3      -1        -2
 5    -3      -1         2
-5    -3       1        -2
 ```
 
#### Time is not a number
为什么如下不能编译？
 
 ```
n := 100
time.Sleep(n * time.Millisecond)

// output
invalid operation: n * time.Millisecond (mismatched types int and time.Duration)
 ```
 
##### 答案
 go语言不存在混合数字类型， 只能用如下类型与time.Duration相乘：
- 另外一个time.Duration
- 无类型的整型常数

下面是三个正确的例子：

```
var n time.Duration = 100
time.Sleep(n * time.Millisecond)

const n = 100
time.Sleep(n * time.Millisecond)

time.Sleep(100 * time.Millisecond)
```
 
#### Index out of range
为什么下面程序会崩溃？

```
a := []int{1, 2, 3}
for i := 1; i <= len(a); i++ {
    fmt.Println(a[i])
}

//output

panic: runtime error: index out of range

goroutine 1 [running]:
main.main()
	../main.go:3 +0xe0
```

##### 答案
最后一轮迭代，i等于len(a)， 但这已经超出了a的极限范围。

#### Unexpected values in range loop
为什么下面程序输出如下？

```
primes := []int{2, 3, 5, 7}
for p := range primes {
    fmt.Println(p)
}

// output

0
1
2
3
```

##### 答案
对于数组， slice， range 输出两个值：
- index
- 相应index对应的数据

如果忽略第二个值，那么只会得到index, 如果需要得到相应位置的值而不是index:

```
primes := []int{2, 3, 5, 7}
for _, p := range primes {
    fmt.Println(p)
}
```

#### Can’t change entries in range loop
为什么下面代码的slice不会更新？

```
s := []int{1, 1, 1}
for _, n := range s {
    n += 1
}
fmt.Println(s)
// Output: [1 1 1]
```

##### 答案
range loop会拷贝相应位置的值到n， 更新n并不能对原值产生影响。
符合预期代码如下：

```
s := []int{1, 1, 1}
for i := range s {
    s[i] += 1
}
fmt.Println(s)
// Output: [2 2 2]
```

#### Iteration variable doesn’t see change in range loop

```
var a [2]int
for _, x := range a {
    fmt.Println("x =", x)
    a[1] = 8
}
fmt.Println("a =", a)

// output

x = 0
x = 0        <- Why isn't this 8?
a = [0 8]
```

##### 答案
range表达式 a只会在loop开始时加载一次。
为了避免这种情况， 迭代使用slice替换即可：

```
var a [2]int
for _, x := range a[:] {
    fmt.Println("x =", x)
    a[1] = 8
}
fmt.Println("a =", a)

// output

x = 0
x = 8
a = [0 8]
```

#### Iteration variables and closures
为什么下面程序输出如下？

```
func main() {
    var wg sync.WaitGroup
    wg.Add(5)
    for i := 0; i < 5; i++ {
        go func() {
            fmt.Print(i)
            wg.Done()
        }()
    }
    wg.Wait()
    fmt.Println()
}

// output

55555
```

##### 答案
存在数据竞争，变量i被6个协程所共享，为了避免这种情况，当开始一个协程时候，通过传参的方式变成局部变量：

```
func main() {
    var wg sync.WaitGroup
    wg.Add(5)
    for i := 0; i < 5; i++ {
        go func(n int) { // Use a local variable.
            fmt.Print(n)
            wg.Done()
        }(i)
    }
    wg.Wait()
    fmt.Println()
}

// output:

40123
```

如果仍然使用闭包，并且避免这种情况，就必须为每一个协程使用一个唯一个变量：

```
func main() {
    var wg sync.WaitGroup
    wg.Add(5)
    for i := 0; i < 5; i++ {
        n := i // Create a unique variable for each closure.
        go func() {
            fmt.Print(n)
            wg.Done()
        }()
    }
    wg.Wait()
    fmt.Println()
}
```

#### No JSON in sight
为什么json.Marshal产生了空的结构对象？

```
type Person struct {
    name string
    age  int
}

p := Person{"Alice", 22}
jsonData, _ := json.Marshal(p)
fmt.Println(string(jsonData))

//output

{}
```

##### 答案
go语言中只有外部可见的对象才能被json序列化：

```
type Person struct {
    Name string // Changed to capital N
    Age  int    // Changed to capital A
}

p := Person{"Alice", 22}

jsonData, _ := json.Marshal(p)
fmt.Println(string(jsonData))

//output

{"Name":"Alice","Age":22}
```

同时可以指定json的关键字标签。

```
type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

p := Person{"Alice", 22}

jsonData, _ := json.Marshal(p)
fmt.Println(string(jsonData))

// output

{"name":"Alice","age":22}
```

#### Is "three" a digit?
为什么匹配数字的正则表达式[0-9]*， 会匹配上字符串？

```
matched, err := regexp.MatchString(`[0-9]*`, "12three45")
fmt.Println(matched) // true
fmt.Println(err)     // nil (regexp is valid)
```

##### 答案
函数regexp.MatchString会做substring的匹配。
如果要做全部的数字匹配[0-9]*， 需要使用开始和结束表达式：
- ^ 会匹配text开头
- $ 会匹配tetx的结尾

```
matched, err := regexp.MatchString(`^[0-9]*$`, "12three45")
fmt.Println(matched) // false
fmt.Println(err)     // nil (regexp is valid)
```

#### nil is not nil
为什么下面的nil不等于nil?

```
func Foo() error {
    var err *os.PathError = nil
    // …
    return err
}

func main() {
    err := Foo()
    fmt.Println(err)        // <nil>
    fmt.Println(err == nil) // false
}
```

仅当interface的值和动态类型都为nil的时候，这个interface才等于nil。
本例中，Foo()返回 [nil, *os.PathError]， 但是和[nil, nil]进行了比较。

你也可以认为interface中为nil的值是有类型的， 没有类型的nil是不等于有类型的nil的，如果转换nil为合适的类型， 就相等了：

```
fmt.Println(err == (*os.PathError)(nil)) // true
```

##### 更好地解释
为了避免使用具有类型的error变量产生问题， 可以如下：

```
func Foo() (err error) {
    // …
    return // err is unassigned and has zero value [nil, nil]
}

func main() {
    err := Foo()
    fmt.Println(err)        // <nil>
    fmt.Println(err == nil) // true
}
```

使用内建的error interface 类型而不是其他类型，来存储和返回error值。
