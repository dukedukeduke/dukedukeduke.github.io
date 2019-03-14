---
layout: post
title:  "golang 基础"
date:   2019-02-03 08:21:02 +0800
comments: true
tags:
- golang
- go语言
---

#### Go 环境变量
- $GOROOT 表示 Go 在你的电脑上的安装位置，它的值一般都是 $HOME/go，当然，你也可以安装在别的地方。
- $GOARCH 表示目标机器的处理器架构，它的值可以是 386、amd64 或 arm。
- $GOOS 表示目标机器的操作系统，它的值可以是 darwin、freebsd、linux 或 windows。
- $GOBIN 表示编译器和链接器的安装位置，默认是 $GOROOT/bin，如果你使用的是 Go 1.0.3 及以后的版本，一般情况下你可以将它的值设置为空，Go 将会使用前面提到的默认值。
- $GOPATH 默认采用和 $GOROOT 一样的值，但从 Go 1.1 版本开始，你必须修改为其它路径。它可以包含多个包含 Go 语言源码文件、包文件和可执行文件的路径，而这些路径下又必须分别包含三个规定的目录：src、pkg 和bin，这三个目录分别用于存放源码文件、包文件和可执行文件。
- $GOMAXPROCS 用于设置应用程序可使用的处理器个数与核数
#### Go 运行时（runtime）
- 尽管 Go 编译器产生的是本地可执行代码，这些代码仍旧运行在 Go 的 runtime（这部分的代码可以在 runtime 包中找到）当中。这个 runtime 类似 Java 和 .NET 语言所用到的虚拟机，它负责管理包括内存分配、垃圾回收（第 10.8 节）、栈处理、goroutine、channel、切片（slice）、map 和反射（reflection）等等
- Go 不需要依赖任何其它文件，它只需要一个单独的静态文件，这样你也不会像使用其它语言一样在各种不同版本的依赖文件之间混淆。
####  构建并运行 Go 程序
- go build 编译并安装自身包和依赖包
- go install 安装自身包和依赖包
#### go fmt
- 这个工具可以将你的源代码格式化成符合官方统一标准的风格，属于语法风格层面上的小型重构\
- 在命令行输入 gofmt –w program.go 会格式化该源文件的代码然后将格式化后的代码覆盖原始内容（如果不加参数 -w则只会打印格式化后的结果而不重写文件）；gofmt -w *.go 会格式化并重写所有 Go 源文件；gofmt map1 会格式化并重写 map1 目录及其子目录下的所有 Go 源文件
#### 关键字与标识符
- 25 个关键字或保留字
- 36 个预定义标识符
#### Go 程序的基本结构和要素
- Go 中的包模型采用了显式依赖关系的机制来达到快速编译的目的，编译器会从后缀名为 .o 的对象文件（需要且只需要这个文件）中提取传递依赖类型的信息。
- 如果 A.go 依赖 B.go，而 B.go 又依赖 C.go：编译 C.go, B.go, 然后是 A.go,为了编译 A.go, 编译器读取的是 B.o 而不是 C.o
- 当你导入多个包时，导入的顺序会按照字母排序。
- 如果包名不是以 . 或 / 开头，如 "fmt" 或者 "container/list"，则 Go 会在全局文件进行查找；如果包名以 ./ 开头，则 Go 会在相对目录中查找；如果包名以 / 开头（在 Windows 下也可以这样使用），则会在系统的绝对路径中查找。
- 当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 private ）
- 函数里的代码（函数体）使用大括号 {} 括起来。
左大括号 { 必须与方法的声明放在同一行，这是编译器的强制规定，否则你在使用 gofmt 时就会出现错误提示
- 一个函数可以拥有多返回值，返回类型之间需要使用逗号分割，并使用小括号 () 将它们括起来
- 使用 type 关键字可以定义你自己的类型，你可能想要定义一个结构体(第 10 章)，但是也可以定义一个已经存在的类型的别名，如：type IZ int
- 每个值都必须在经过编译后属于某个类型（编译器必须能够推断出所有值的类型），因为 Go 语言是一种静态类型语言
- Go 程序的执行（程序启动）顺序如下：
1. 按顺序导入所有被 main包引用的其它包，然后在每个包中执行如下流程：
2. 如果该包又导入了其它的包，则从第一步开始递归执行，但是每个包只会被导入一次。
3. 然后以相反的顺序在每个包中初始化常量和变量，如果该包含有 init 函数的话，则调用该函数。
4. 在完成这一切之后，main 也执行同样的过程，最后调用 main 函数开始执行程序。
- 由于 Go 语言不存在隐式类型转换，因此所有的转换都必须显式说明，就像调用一个函数一样（类型在这里的作用可以看作是一种函数）：valueOfTypeB = typeB(valueOfTypeA)
#### 常量
- 常量使用关键字 const 定义，用于存储不会改变的数据, const Pi = 3.14159;const b string = "abc"
- 错误的做法：const c2 = getNumber() // 引发构建错误: getNumber() used as value
- 在编译期间自定义函数均属于未知，因此无法用于常量的赋值，但内置函数可以使用，如：len()
- 数字型的常量是没有大小和符号的，并且可以使用任何精度而不会导致溢出
- 反斜杠 \ 可以在常量表达式中作为多行的连接符使用
- 常量还可以用作枚举：

```
const (
    Unknown = 0
    Female = 1
    Male = 2
)
```

- iota 可以被用作枚举值,第一个 iota 等于 0，每当 iota 在新的一行被使用时，它的值都会自动加 1；所以 a=0, b=1, c=2

```
const (
    a = iota
    b = iota
    c = iota
)
```

#### 变量
- var identifier type
- 举例

```
var a int
var b bool
var str string  
```

你也可以改写成这种形式：

```
var (
    a int
    b bool
    str string
)
```

- 当一个变量被声明之后，系统自动赋予它该类型的零值：int 为 0，float 为 0.0，bool 为 false，string 为空字符串，指针为 nil。记住，所有的内存在 Go 中都是经过初始化的。
-  Go 编译器的智商已经高到可以根据变量的值来自动推断其类型，这有点像 Ruby 和 Python 这类动态语言，只不过它们是在运行时进行推断，而 Go 是在编译时就已经完成推断过程。因此，你还可以使用下面的这些形式来声明及初始化变量：

```
var a = 15
var b = false
var str = "Go says hello to the world!"
```

- 不过自动推断类型并不是任何时候都适用的，当你想要给变量的类型并不是自动推断出的某种类型时，你还是需要显式指定变量的类型, 比如： var a这种语法是不正确的，因为编译器没有任何可以用于自动推断类型的依据。

- 当你在函数体内声明局部变量时，应使用简短声明语法 :=，例如：a := 1
- 像 int、float、bool 和 string 这些基本类型都属于值类型，使用这些类型的变量直接指向存在内存中的值,像数组（第 7 章）和结构（第 10 章）这些复合类型也是值类型, 值类型的变量的值存储在栈中。
- 你可以通过 &i 来获取变量 i 的内存地址。
- 更复杂的数据通常会需要使用多个字，这些数据一般使用引用类型保存。一个引用类型的变量 r1 存储的是 r1 的值所在的内存地址（数字），或内存地址中第一个字所在的位置，这个内存地址为称之为指针， 当使用赋值语句 r2 = r1 时，只有引用（地址）被复制。
- 如果在相同的代码块中，我们不可以再次对于相同名称的变量使用初始化声明
- 全局变量是允许声明但不使用， 局部变量则不行。
- 空白标识符 _ 也被用于抛弃值，如值 5 在：_, b = 5, 7 中被抛弃。
- 变量除了可以在全局声明中初始化，也可以在 init 函数中初始化。这是一类非常特殊的函数，它不能够被人为调用，而是在每个包完成初始化后自动执行，并且执行优先级比 main 函数高。
- 每一个源文件都可以包含且只包含一个 init 函数。初始化总是以单线程执行，并且按照包的依赖关系顺序执行。
- init 函数一个可能的用途是在开始执行程序之前对数据进行检验或修复，以保证程序状态的正确性。另外，init 函数也经常被用在当一个程序开始之前调用后台执行的 goroutine。
- 字符串是 UTF-8 字符的一个序列（当字符为 ASCII 码时则占用 1 个字节，其它字符根据需要占用 2-4 个字节）。
- 和 C/C++不一样，Go中的字符串是根据长度限定，而非特殊字符\0。
#### time 包
- time 包为我们提供了一个数据类型time.Time（作为值使用）以及显示和测量时间和日期的功能函数
- 当前时间可以使用 time.Now() 获取，或者使用 t.Day()、t.Minute()等等来获取时间的一部分；你甚至可以自定义时间格式化字符串，例如：fmt.Printf("%02d.%02d.%4d\n", t.Day(), t.Month(), t.Year()) 将会输出 21.07.2011
#### 指针
- 你不能进行指针运算
- Go 语言的取地址符是 &，放到一个变量前使用就会返回相应变量的内存地址
- 地址可以存储在一个叫做指针的特殊数据类型中
- var intP *int 然后使用 intP = &i1 是合法的，此时 intP 指向 i1
- 一个指针变量可以指向任何一个值的内存地址 它指向那个值的内存地址，在 32 位机器上占用 4 个字节，在 64 位机器上占用 8 个字节，并且与它所指向的值的大小无关
- 当一个指针被定义后没有分配到任何变量时，它的值为 nil
- Go 语言和 C、C++ 以及 D 语言这些低级（系统）语言一样，都有指针的概念。但是对于经常导致 C 语言内存泄漏继而程序崩溃的指针运算（所谓的指针算法，如：pointer+2，移动指针指向字符串的字节数或数组的某个位置）是不被允许的。
- 指针的一个高级应用是你可以传递一个变量的引用（如函数的参数），这样不会传递变量的拷贝。指针传递是很廉价的，只占用 4 个或 8 个字节。
#### if-else 结构

```
if condition1 {
    // do something 
} else if condition2 {
    // do something else    
}else {
    // catch-all or default
}

```

#### 函数返回值的错误表示值
Go 语言的函数经常使用两个返回值来表示执行是否成功：返回某个值以及 true 表示成功；返回零值（或 nil）和 false 表示失败（第 4.4 节）。当不使用 true 或 false 的时候，也可以使用一个 error 类型的变量来代替作为第二个返回值：成功执行的话，error 的值为 nil，否则就会包含相应的错误信息（Go 语言中的错误类型为 error: var err error，我们将会在第 13 章进行更多地讨论）
#### switch

```
switch var1 {
    case val1:
        ...
    case val2:
        ...
    default:
        ...
}
```

#### for 结构

```
for 初始化语句; 条件语句; 修饰语句 {}
```

- for-range 结构
它可以迭代任何一个集合（包括数组和 map，详见第 7 和 8 章）。语法上很类似其它语言中 foreach 语句，但您依旧可以获得每次迭代所对应的索引。一般形式为：for ix, val := range coll { }。
要注意的是，val 始终为集合中对应索引的值拷贝，因此它一般只具有只读性质，对它所做的任何修改都不会影响到集合中原有的值（译者注：如果 val 为指针，则会产生指针的拷贝，依旧可以修改集合中的原值）。
#### 函数
- 函数被调用的时候，这些实参将被复制（简单而言）然后传递给被调用函数。函数一般是在其他函数里面被调用的，这个其他函数被称为调用函数（calling function）
- 函数重载（function overloading）指的是可以编写多个同名函数，只要它们拥有不同的形参与/或者不同的返回值，在 Go 里面函数重载是不被允许的。
- 函数也可以以申明的方式被使用，作为一个函数类型，就像：

```
type binOp func(int, int) int
```

- 函数是一等值（first-class value）：它们可以赋值给变量，就像 add := binOp 一样。
- Go 默认使用按值传递来传递参数，也就是传递参数的副本。函数接收参数副本之后，在使用变量的过程中可能对副本的值进行更改，但不会影响到原来的变量
- 如果你希望函数可以直接修改参数的值，而不是对参数的副本进行操作，你需要将参数的地址（变量名前面添加&符号，比如 &variable）传递给函数，这就是按引用传递，比如 Function(&arg1)，此时传递给函数的是一个指针
- 几乎在任何情况下，传递指针（一个32位或者64位的值）的消耗都比传递副本来得少。
- 在函数调用时，像切片（slice）、字典（map）、接口（interface）、通道（channel）这样的引用类型都是默认使用引用传递（即使没有显示的指出指针）。
- 传递指针给函数不但可以节省内存（因为没有复制变量的值），而且赋予了函数直接修改外部变量的能力，所以被修改的变量不再需要使用 return 返回.
- 如果函数的最后一个参数是采用 ...type 的形式，那么这个函数就可以处理一个变长的参数，这个长度可以为 0，这样的函数称为变参函数。

```
func myFunc(a, b, arg ...int) {}
```

- 在 Greeting 函数中，变量 who 的值为 []string{"Joe", "Anna", "Eileen"}

```
func Greeting(prefix string, who ...string)
Greeting("hello:", "Joe", "Anna", "Eileen")
```

- 关键字 defer 允许我们推迟到函数返回之前（或任意位置执行 return 语句之后）一刻才执行某个语句或函数（为什么要在返回之后才执行这些语句？因为 return 语句同样可以包含一些操作，而不是单纯地返回某个值）。
- 函数可以作为其它函数的参数进行传递，然后在其它函数内调用执行，一般称之为回调
- 闭包：在程序 function_return.go 中我们将会看到函数 Add2 和 Adder 均会返回签名为 func(b int) int 的函数：

```
func Add2() (func(b int) int)
func Adder(a int) (func(b int) int)
```

- 通过内存缓存来提升性能：将第 n 个数的值存在数组中索引为 n 的位置（详见第 7 章），然后在数组中查找是否已经计算过，如果没有找到，则再进行计算。
#### 数组
- var identifier [len]type
- 数组长度必须是一个常量表达式，并且必须是一个非负整数
- 数组元素可以通过 索引（位置）来读取（或者修改）
- var identifier [len]type
- for 遍历数组：在这里i也是数组的索引。

```
for i,_:= range arr1 {
...
}
```

- Go 语言中的数组是一种 值类型（不像 C/C++ 中是指向首元素的指针），所以可以通过 new() 来创建： var arr1 = new([5]int)。
那么这种方式和 var arr2 [5]int 的区别是什么呢？arr1 的类型是 *[5]int，而 arr2的类型是 [5]int
- 两个数组就有了不同的值，在赋值后修改 arr2 不会对 arr1 生效。

```
arr2 := arr1
arr2[2] = 100
```

- 所以在函数中数组作为参数传入时，如 func1(arr2)，会产生一次数组拷贝，func1 方法不会修改原始的数组 arr2.如果你想修改原数组，那么 arr2 必须通过&操作符以引用方式传过来，例如 func1(&arr2）
- 切片

```
切片（slice）是对数组一个连续片段的引用,这个片段可以是整个数组，或者是由起始和终止索引标识的一些项的子集。需要注意的是，终止索引标识的项不包括在切片内。切片提供了一个相关数组的动态窗口。切片提供了计算容量的函数 cap() 可以测量切片最长可以达到多少：它等于切片的长度 + 数组除切片之外的长度。因为切片是引用，所以它们不需要使用额外的内存并且比使用数组更有效率，所以在 Go 代码中 切片比数组更常用.声明切片的格式是： var identifier []type（不需要说明长度）.
```

#### map
- 声明

```
var map1 map[keytype]valuetype
var map1 map[string]int
```

- key 可以是任意可以用 == 或者 != 操作符比较的类型，比如 string、int、float。所以数组、切片和结构体不能作为 key，但是指针和接口类型可以。如果要用结构体作为 key 可以提供 Key() 和 Hash() 方法，这样可以通过结构体的域计算出唯一的数字或者字符串的 key。
- value 可以是任意类型的
- map 传递给函数的代价很小：在 32 位机器上占 4 个字节，64 位机器上占 8 个字节，无论实际上存储了多少数据。通过 key 在 map 中寻找值是很快的，比线性查找快得多，但是仍然比从数组和切片的索引中直接读取要慢 100 倍；所以如果你很在乎性能的话还是建议用切片来解决问题。
- key1 对应的值可以通过赋值符号来设置为 val1：map1[key1] = val1
- map 是 引用类型 的： 内存用 make 方法来分配。map 的初始化：

```
var map1[keytype]valuetype = make(map[keytype]valuetype)。
或者简写为：
map1 := make(map[keytype]valuetype)。
```

- val1, isPresent = map1[key1]
isPresent 返回一个 bool 值：如果 key1 存在于 map1，val1 就是 key1 对应的 value 值，并且 isPresent为true；如果 key1 不存在，val1 就是一个空值，并且 isPresent 会返回 false。
- for range

```
for key, value := range map1 {
    ...
}

```

- map 默认是无序的，不管是按照 key 还是按照 value 默认都不排序。如果你想为 map 排序，需要将 key（或者 value）拷贝到一个切片，再对切片排序（使用 sort 包），然后可以使用切片的 for-range 方法打印出所有的 key 和 value。
#### 常见包
- regexp
- sync包
- 如果你要在你的应用中使用一个或多个外部包，首先你必须使用 go install在你的本地机器上安装它们。
假设你想使用 http://codesite.ext/author/goExample/goex 这种托管在 Google Code、GitHub 和 Launchpad等代码网站上的包。你可以通过如下命令安装：

```
go install codesite.ext/author/goExample/goex
将一个名为 codesite.ext/author/goExample/goex 的 map 安装在 $GOROOT/src/ 目录下。
```

- 程序的执行开始于导入包，初始化 main 包然后调用 main 函数。
一个没有导入的包将通过分配初始值给所有的包级变量和调用源码中定义的包级 init 函数来初始化。一个包可能有多个 init 函数甚至在一个源码文件中。它们的执行是无序的。这是最好的例子来测定包的值是否只依赖于相同包下的其他值或者函数。
init 函数是不能被调用的。
导入的包在包自身初始化前被初始化,而一个包在程序执行中只能初始化一次。
- 常用包

```
MySQL(GoMySQL), PostgreSQL(go-pgsql), MongoDB (mgo, gomongo), CouchDB (couch-go), ODBC (godbcl), Redis (redis.go) and SQLite3 (gosqlite) database drivers
SDL bindings
Google's Protocal Buffers(goprotobuf)
XML-RPC(go-xmlrpc)
Twitter(twitterstream)
OAuth libraries(GoAuth)
```

#### struct
- 结构体定义的一般方式如下：

```
type identifier struct {
    field1 type1
    field2 type2
    ...
}
```

- 使用 new 函数给一个新的结构体变量分配内存，它返回指向已分配内存的指针：var t *T = new(T)，如果需要可以把这条语句放在不同的行（比如定义是包范围的，但是分配却没有必要在开始就做）。
- 写这条语句的惯用方法是：t := new(T)，变量 t是一个指向 T的指针，此时结构体字段的值是它们所属类型的零值。声明var t T 也会给 t分配内存，并零值化内存，但是这个时候t是类型T。在这两种方式中，t通常被称做类型T的一个实例（instance）或对象（Object）。

```
var t *T
t = new(T)
```

- 就像在面向对象语言所作的那样，可以使用点号符给字段赋值：structname.fieldname = value。同样的，使用点号符可以获取结构体字段的值：structname.feldname。在 Go 语言中这叫选择器（selector）。无论变量是一个结构体类型还是一个结构体类型指针，都使用同样的 选择器符（selector-notation） 来引用结构体的字段
- 在 Go 中，类型的代码和绑定在它上面的方法的代码可以不放置在一起，它们可以存在在不同的源文件，唯一的要求是：它们必须是同一个包的
- 定义结构体方法

```
type Person struct {
    firstName string
    lastName  string
}

func (p *Person) SetFirstName(newName string) {
    p.firstName = newName
}

p := new(Person)
p.SetFirstName("Eric")
```

#### interface
- 接口提供了一种方式来说明对象的行为：如果谁能搞定这件事，它就可以用在这儿。
- 接口定义了一组方法（方法集），但是这些方法不包含（实现）代码：它们没有被实现（它们是抽象的）。接口里也不能包含变量。

```
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
```

#### 错误处理
- Go 有一个预先定义的 error 接口类型

```
type error interface { 
    Error() string
}
```

- 任何时候当你需要一个新的错误类型，都可以用 errors（必须先 import）包的 errors.New 函数接收合适的错误信息来创建，像下面这样：

```
err := errors.New(“math - square root of negative number”)
```

```
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New (“math - square root of negative number”)
    }
   // implementation of Sqrt
}
if f, err := Sqrt(-1); err != nil {
    fmt.Printf(“Error: %s\n”, err)
}

```

- 从错误中恢复

```
func protect(g func()) {
    defer func() {
        log.Println(“done”)
        // Println executes normally even if there is a panic 
        if err := recover(); err != nil {
        log.Printf(“run time panic: %v”, err)
        }
    }()
    log.Println(“start”)
    g() //   possible runtime-error
}
```

#### 协程
- 在 Go 中，应用程序并发处理的部分被称作 goroutines（协程），它可以进行更有效的并发运算。在协程和操作系统线程之间并无一对一的关系：协程是根据一个或多个线程的可用性，映射（多路复用，执行于）在他们之上的；协程调度器在 Go运行时很好的完成了这个工作。协程工作在相同的地址空间中，所以共享内存的方式一定是同步的；这个可以使用 sync 包来实现（参见第 9.3 节），不过我们很不鼓励这样做：Go 使用 channels 来同步协程（可以参见第 14.2 节等章节）
- 在 gc 编译器下（6g 或者 8g）你必须设置 GOMAXPROCS 为一个大于默认值 1 的数值来允许运行时支持使用多于 1 个的操作系统线程，所有的协程都会共享同一个线程除非将 GOMAXPROCS 设置为一个大于 1 的数。当 GOMAXPROCS 大于 1 时，会有一个线程池管理许多的线程。通过 gccgo 编译器 GOMAXPROCS 有效的与运行中的协程数量相等。假设 n 是机器上处理器或者核心的数量。如果你设置环境变量 GOMAXPROCS>=n，或者执行 runtime.GOMAXPROCS(n)，接下来协程会被分割（分散）到 n 个处理器上。更多的处理器并不意味着性能的线性提升。有这样一个经验法则，对于 n 个核心的情况设置 GOMAXPROCS 为 n-1 以获得最佳性能，也同样需要遵守这条规则：协程的数量 > 1 + GOMAXPROCS > 1。
- 在其他语言中，比如 C#，Lua 或者 Python 都有协程的概念。这个名字表明它和 Go协程有些相似，不过有两点不同：
1. Go 协程意味着并行（或者可以以并行的方式部署），协程一般来说不是这样的
2. Go 协程通过通道来通信；协程通过让出和恢复操作来通信
Go 协程比协程更强大，也很容易从协程的逻辑复用到 Go 协程。
- 通道

```
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan string)

    go sendData(ch)
    go getData(ch)  

    time.Sleep(1e9)
}

func sendData(ch chan string) {
    ch <- "Washington"
    ch <- "Tripoli"
    ch <- "London"
    ch <- "Beijing"
    ch <- "Tokio"
}

func getData(ch chan string) {
    var input string
    // time.Sleep(1e9)
    for {
        input = <-ch
        fmt.Printf("%s ", input)
    }
}
```

- 默认情况下，通信是同步且无缓冲的：在有接受者接收数据之前，发送不会结束。可以想象一个无缓冲的通道在没有空间来保存数据的时候：必须要一个接收者准备好接收通道的数据然后发送者可以直接把数据发送给接收者。所以通道的发送/接收操作在对方准备好之前是阻塞的：
1）对于同一个通道，发送操作（协程或者函数中的），在接收者准备好之前是阻塞的：如果ch中的数据无人接收，就无法再给通道传入其他数据：新的输入无法在通道非空的情况下传入。所以发送操作会等待 ch 再次变为可用状态：就是通道值被接收时（可以传入变量）。
2）对于同一个通道，接收操作是阻塞的（协程或函数中的），直到发送者可用：如果通道中没有数据，接收者就阻塞了。
- 带缓冲的通道

```
buf := 100
ch1 := make(chan string, buf)
buf 是通道可以同时容纳的元素（这里是 string）个数
```




