## 常用链接

- [https://golang.google.cn/](https://golang.google.cn/)
- [Go知识图谱](https://www.processon.com/view/link/5a9ba4c8e4b0a9d22eb3bdf0#map)
- [Golang中国（学习网站）](https://www.qfgolang.com/)
- [Go语言圣经](https://books.studygolang.com/gopl-zh/)
- [Go中文说明文档](http://www.topgoer.com/)
- [Go李文周博客](https://www.liwenzhou.com/posts/Go/go_menu/)
- [地鼠文档](https://www.topgoer.cn/) (Go 大合集)
- Go语言比较好的资料：《Go入门指南》 —> [Go高阶基础](https://www.aliyundrive.com/s/11nP7waj6nB) —> [Go设计与实现](https://draveness.me/golang/)

## 安装

[Go 安装教程](https://zhuanlan.zhihu.com/p/265775951)

## IDEA 开发Go

[intelliJ idea安装go开发环境 并 搭建go项目 打包](https://blog.csdn.net/chushoutaizhong/article/details/82220419)

idea 2019.3.1 的go插件有问题，不能设置GOROOT，更换为idea 2021.3.1即可。

[IntelliJ lDEA 2021.3.1激活破解图文教程（亲测有用，永久激活)](https://www.cnblogs.com/geekxh/p/15754800.html)

## 问题

### Go错误：$GOPATH/go.mod exists but should not

产生原因：开启模块支持后，并不能与$GOPATH共存,所以把项目从$GOPATH中移出即可

### go  get 下载依赖失败

具体·报错如下：

```go
go get: module github.com/julienschmidt/httprouter: Get "https://proxy.golang.org/github.com/julienschmidt/httprouter/@v/list": dial tcp 172.217.1
60.81:443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established conn
ection failed because connected host has failed to respond.
```

解决：

把它关掉

```bash
go env -w GOSUMDB=off
```

代理推荐

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

然后即可

## Go基础

### 特点

- Go 语言在这 3 个条件之间做到了最佳的平衡：快速编译，高效执行，易于开发。
- Go 语言中仍有指针的存在，但并不允许进行指针运算。
- 通过 goroutine 这种轻量级线程的概念，然后通过 channel 来实现各个 goroutine 之间的通信，对于网络通信、并发和并行编程的极佳支持
- Go语言通过减少关键字的数量（25 个）来简化编码过程中的混乱和复杂度
- Go 语言没有类和继承的概念，但是它通过接口（interface）**的概念来实现多态性**

### 语法

#### main

- 应⽤用程序⼊入⼝口
- Go 中 main 函数不不⽀支持任何返回值
- 通过 os.Exit 来返回状态
- main 函数不不⽀支持传⼊入参数：func main~~(arg []string)~~
- 在程序中直接通过 os.Args 获取命令⾏行行参数

#### 变量赋值

- 赋值可以进⾏行行⾃自动类型推断 
- 在⼀一个赋值语句句中可以对多个变量量进⾏行行同时赋值 

` _  ` 本身就是一个特殊的标识符，被称为空白标识符。它可以像其他标识符那样用于变量的声明或赋值（任何类型都可以赋值给它），但任何赋给这个标识符的值都将被抛弃，因此这些值不能在后续的代码中使用，也不可以使用这个标识符作为变量对其它变量进行赋值或运算。

你也可以改写成这种形式：

```go
var ( 
    a int 
    b bool 
    str string 
)
```

可以通过 `&i` 来获取变量 `i ` 的内存地址。

在 Go 语言中，指针属于引用类型，其它的引用类型还包括 slices，maps和 channel。

**使用 := 赋值操作符，这是使用变量的首选形式，但是它只能被用在函数体内，而不可以用于全局变量的声明与赋值**。

如果你想要交换两个变量的值，则可以简单地使用  `a, b = b, a `

#### 常量定义

**存储在常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型。**

快速设置连续值

```go
const (
  Monday = iota + 1
  Tuesday
  Wednesday
  Thursday
  Friday
  Saturday
  Sunday
)

const (
  Open = 1 << iota
  Close
  Pending
)
```

反斜杠 `\` 可以在常量表达式中作为多行的连接符使用:

```GO
const Ln2 = 0.693147180559945309417232121458\
            176568075500134360255254120680009
```



#### package

1. 基本复用模块单元 ，以⾸首字母大写来表明可被包外代码访问
2. 代码的 package 可以和所在的目录不一致 
3. 同一目录里的 Go 代码的 package 要保持一致
4. 通过 `go get` 来获取远程依赖，`go get -u` 强制从⽹网络更更新远程依赖
5. 注意代码在 GitHub 上的组织形式，以适应 `go get`，直接以代码路路径开始，不不要有 src

#### init 方法

- 在 main 被执行前，所有依赖的 package 的 init 方法都会被执行
- 不同包的 init 函数按照包导入的依赖关系决定执行顺序
- 每个包可以有多个 init 函数
- 包的每个源文件也可以有多个 init 函数，这点比较特殊

### 基本数据类型

```go
bool
string
int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr
byte // alias for uint8
rune // alias for int32,represents a Unicode code point
float32 float64
complex64 complex128
```

Go 语⾔言不不允许隐式类型转换

别名和原有类型也不不能进⾏行行隐式类型转换

```go
// 类型转换 （不能隐式转换）
func TestImplicit(t *testing.T) {
	var a int32 = 1
	var b int64
	b = int64(a)
	var c MyInt
	c = MyInt(b)
	t.Log(a, b, c)
}
```



#### 指针类型

不不⽀支持指针运算

```go
// 指针类型
func TestPoint(t *testing.T) {
	a := 1
	aPtr := &a
	// 指针不能进行运算
	//aPtr = aPtr + 1
	t.Log(a, aPtr)
	// 输出其 类型
	t.Logf("%T %T", a, aPtr)
}
```



### 运算符

#### 算术运算符

![算术运算符](https://cos.duktig.cn/typora/202202082009887.png)

#### ⽐比较运算符

![image-20220208200957392](https://cos.duktig.cn/typora/202202082010449.png)

**⽤用 == ⽐比较数组** 

- 相同维数且含有相同个数元素的数组才可以⽐比较
- 每个元素都相同的才相等

```go
func TestCompareArray(t *testing.T) {
	a := [...]int{1, 2, 3, 4}
	b := [...]int{1, 3, 2, 4}
	//c := [...]int{1, 2, 3, 4, 5}
	d := [...]int{1, 2, 3, 4}
	t.Log(a == b)
	//t.Log(a == c)
	t.Log(a == d)
}
```



#### 逻辑运算符

![image-20220208201104058](https://cos.duktig.cn/typora/202202082011009.png)

#### 位运算符

![image-20220208201141925](https://cos.duktig.cn/typora/202202082011727.png)

**&^ 按位置零**

- 1 &^ 0 --  1
- 1 &^ 1 --  0
- 0 &^ 1 --  0
- 0 &^ 0 --  0

### 条件和循环

#### 循环

Go 语⾔言仅⽀支持循环关键字 `for`

#### if

⽀支持变量量赋值：

```go
if  var declaration;  condition { 
    // code to be executed if condition is true
}
```

#### switch

1. 条件表达式不不限制为常量量或者整数； 
2. 单个 case 中，可以出现多个结果选项, 使⽤用逗号分隔； 
3. 与 C 语⾔言等规则相反，Go 语⾔言不不需要⽤用break来明确退出⼀一个 case； 
4. 可以不不设定 switch 之后的条件表达式，在此种情况下，整个 switch 结构与多个 if…else… 的逻辑作⽤用等同

```go
func TestSwitchOS(t *testing.T) {
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
		//break
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
}

// switch 当 if 使用
func TestSwitchAndIf(t *testing.T) {
	Num := rand.Intn(10)
	fmt.Println(Num)
	switch {
	case 0 <= Num && Num <= 3:
		fmt.Printf("0-3")
	case 4 <= Num && Num <= 6:
		fmt.Printf("4-6")
	case 7 <= Num && Num <= 9:
		fmt.Printf("7-9")
	}
}

// 多值 Switch
func TestSwitchMultiCase(t *testing.T) {
	for i := 0; i < 5; i++ {
		switch i {
		case 0, 2:
			t.Log("Even")
		case 1, 3:
			t.Log("Odd")
		default:
			t.Log("it is not 0-3")
		}
	}
}
```

### 集合

#### 数组

数组的声明

```go
var a [3]int //声明并初始化为默认零值
a[0] = 1
b := [3]int{1, 2, 3}           //声明同时初始化
c := [2][2]int{{1, 2}, {3, 4}} //多维数组初始化
d := [...]int{1, 2, 3, 4, 5} //不不指定元素个数
```

数组元素的遍历

```go
func TestTravelArray(t *testing.T) {
	//不不指定元素个数
	a := [...]int{1, 2, 3, 4, 5}

	for i := 0; i < len(a); i++ {
		t.Log(a[i])
	}

	for idx /*索引*/, elem /*元素*/ := range a {
		fmt.Println(idx, elem)
	}
}
```

数组的截取

```

func TestArraySection(t *testing.T) {
   arr := [...]int{1, 2, 3, 4, 5}
   arr_sec := arr[3:]
   t.Log(arr_sec)
}
```

#### 切片

和数组是相似的，但其是 **可变长的**。声明时，不用确定长度。

**切片只能和nil比较**

切⽚片内部结构

![image-20220208204755924](https://cos.duktig.cn/typora/202202082047104.png)

**切⽚声明**

```go
var s0 []int
s0 = append(s0, 1)
s := []int{}
s1 := []int{1, 2, 3}
s2 := make([]int, 2, 4) 
/*[]type, len, cap
  其中len个元素会被初始化为默认零值，未初始化元素不不可以访问
 */
```

**切片扩容**

```go
// 切片 扩容 —— 达到容量时，2倍扩容
func TestSliceGrowing(t *testing.T) {
	var s []int
	for i := 0; i < 10; i++ {
		s = append(s, i)
		t.Log(len(s), cap(s))
	}
}
```

**切⽚片共享存储结构**

![image-20220208210810569](https://cos.duktig.cn/typora/202202082108055.png)

#### map

**Map 声明**

```go
m := map[string]int{"one": 1, "two": 2, "three": 3}
m1 := map[string]int{}
m1["one"] = 1
m2 := make(map[string]int, 10 /*Initial Capacity*/)
//为什什么不不初始化len？	
```

**Map 元素的访问**

在访问的 Key 不不存在时，仍会返回零值，不不能通过返回 nil 来判断元素是否存在

```go
if v, ok := m["four"]; ok {
  t.Log("four", v)
} else {
  t.Log("Not existing")
}
```

**遍历**

```go
func TestTravelMap(t *testing.T) {
	m1 := map[int]int{1: 1, 2: 4, 3: 9}
	for k, v := range m1 {
		t.Log(k, v)
	}
}
```

### 字符串

1. string 是数据类型，不不是引⽤用或指针类型 ，其默认的初始化值为空字符串串，⽽而不是 `nil`
2. string 是只读的**不可变的byte slice**，len 函数可以它所包含的 byte 数
3. string 的 byte 数组可以存放任何数据

```go
// 字符串
func TestString(t *testing.T) {
	// 默认初始化为 空字符串 " "
	var s string
	t.Log("*" + s + "*") //初始化零值是“”
	t.Log(len(s))
    t.Log(s == "")
}

func TestString2(t *testing.T) {
	var s string
	t.Log(s) //初始化为默认零值“”
	s = "hello"
	t.Log(len(s))
	//s[1] = '3' //string是不可变的byte slice
	//s = "\xE4\xB8\xA5" //可以存储任何二进制数据
	s = "\xE4\xBA\xBB\xFF"
	t.Log(s)
	t.Log(len(s))
	s = "中"
	t.Log(len(s)) //是byte数

	c := []rune(s)
	t.Log(len(c))
	//	t.Log("rune size:", unsafe.Sizeof(c[0]))
	t.Logf("中 unicode %x", c[0])
	t.Logf("中 UTF8 %x", s)
}

```



常⽤用字符串串函数：

- [strings 包](https://golang.org/pkg/strings/) 
- [strconv 包](https://golang.org/pkg/strconv/) 

### 函数

Go 默认使用按值传递来传递参数，也就是传递参数的副本。函数接收参数副本之后，在使用变量的过程中可能对副本的值进行更改，但不会影响到原来的变量，比如 `Function(arg1)`。

如果你希望函数可以直接修改参数的值，而不是对参数的副本进行操作，你需要将参数的地址（变量名前面添加&符号，比如 &variable）传递给函数，这就是按引用传递，比如 `Function(&arg1)`，此时传递给函数的是一个指针。如果传递给函数的是一个指针，指针的值（一个地址）会被复制，但指针的值所指向的地址上的值不会被复制；我们可以通过这个指针的值来修改这个值所指向的地址上的值。（**译者注：指针也是变量类型，有自己的地址和值，通常指针的值指向一个变量的地址。所以，按引用传递也是按值传递。**）

几乎在任何情况下，传递指针（一个32位或者64位的值）的消耗都比传递副本来得少。

**在函数调用时，像切片（slice）、字典（map）、接口（interface）、通道（channel）这样的引用类型都是默认使用引用传递（即使没有显式的指出指针）。**

#### 函数特性

1. 可以有多个返回值
2. 所有参数都是值传递：slice，map，channel 会有传引⽤用的错觉
3. 函数可以作为变量量的值
4. 函数可以作为参数和返回值

```go
// 返回两个随机数
func returnMultiValues() (int, int) {
   return rand.Intn(10), rand.Intn(20)
}

// 功能：计算函数执行时间
// param：函数类型
// return：输入什么，还返回什么 函数类型
func timeSpent(inner func(op int) int) func(op int) int {
   return func(n int) int {
      start := time.Now()
      ret := inner(n)
      fmt.Println("time spent:", time.Since(start).Seconds())
      return ret
   }
}

func slowFun(op int) int {
   time.Sleep(time.Second * 1)
   return op
}

func TestFn(t *testing.T) {
   a, _ := returnMultiValues()
   t.Log(a)
   tsSF := timeSpent(slowFun)
   t.Log(tsSF(10))
}
```

#### 可变参数

```go
// 可变长参数
func Sum(ops ...int) int {
   ret := 0
   for _, op := range ops {
      ret += op
   }
   return ret
}

func TestVarParam(t *testing.T) {
   t.Log(Sum(1, 2, 3, 4))
   t.Log(Sum(1, 2, 3, 4, 5))
}
```



#### defer 延迟函数

关键字 defer 允许我们推迟到函数返回之前（或任意位置执行 `return` 语句之后）一刻才执行某个语句或函数（为什么要在返回之后才执行这些语句？因为 `return` 语句同样可以包含一些操作，而不是单纯地返回某个值）。

关键字 defer 的用法类似于面向对象编程语言 Java 和 C# 的 `finally` 语句块，它一般用于释放某些已分配的资源。

```go
func Clear() {
   fmt.Println("Clear resources.")
}

// 延迟执行函数 defer
func TestDefer(t *testing.T) {
   // 延迟执行
   defer Clear()
   fmt.Println("Start")
   // 中断，defer仍然执行
   panic("err")
}
```



### 结构（struct）

Go 通过类型别名（alias types）和结构体的形式支持用户自定义类型，或者叫定制类型。一个带属性的结构体试图表示一个
现实世界中的实体。结构体是复合类型（composite types），当需要定义一个类型，它由一系列属性组成，每个属性都有自
己的类型和值的时候，就应该使用结构体，它把数据聚集在一起。然后可以访问这些数据，就好像它是一个独立实体的一部
分。结构体也是值类型，因此可以通过 new 函数来创建。

组成结构体类型的那些数据称为 字段（ﬁelds）。每个字段都有一个类型和一个名字；在一个结构体中，字段名字必须是唯一
的。

结构体定义的一般方式如下：

```go
type identifier struct { 
    field1 type1 
    field2 type2 
    ... 
}
```

结构体的字段可以是任何类型，甚至是结构体本身。

使用 **new 函数**给一个新的结构体变量分配内存，它返回指向已分配内存的指针：  `var t *T = new(T) `

惯用方法是：  `t := new(T) ` ，变量  t  是一个指向  T  的指针，此时结构体字段的值是它们所属类型的零值。

在 Go 语言中这叫 选择器（selector）。无论变量是一个结构体类型还是一个结构体类型指针，都使用同样的 选择器符
（selector-notation） 来引用结构体的字段：

```go
type myStruct struct { i int } 
var v myStruct    // v是结构体类型变量
var p *myStruct   // p是指向一个结构体类型变量的指针 
v.i 
p.i
```



初始化一个结构体实例（一个结构体字面量：struct-literal）的更简短和惯用的方式如下：

```go
ms := &struct1{10, 15.5, "Chris"} 
// 此时ms的类型是 *struct1
```

或者：

```go
var ms struct1 
ms = struct1{10, 15.5, "Chris"}
```

混合字面量语法（composite literal syntax）  `&struct1{a, b, c}`  是一种简写，底层仍然会调用  `new ()`  ，这里值的顺序必须按照字段顺序来写。在下面的例子中能看到可以通过在值的前面放上字段名来初始化字段的方式。表达式 `new(Type)`  和  `&Type{}`  是等价的。

#### 工厂方法创建结构体

```go
type File struct { 
    fd      int     // 文件描述符 
    name    string  // 文件名 
}
```

下面是这个结构体类型对应的工厂方法，它返回一个指向结构体实例的指针：

```go
func NewFile(fd int, name string) *File { 
    if fd < 0 { 
        return nil 
    } 
 
    return &File{fd, name} 
}
```

然后这样调用它：

```go
f := NewFile(10, "./test.txt")
```

在 Go 语言中常常像上面这样在工厂方法里使用初始化来简便的实现构造函数。

如果  File  是一个结构体类型，那么表达式  `new(File)`  和  `&File{}`  是等价的。

如果想知道结构体类型T的一个实例占用了多少内存，可以使用：  `size := unsafe.Sizeof(T{})`  。

#### new() 和 make()

可以使用  make()  的三种类型：slices  /  maps / channels

### 方法（method）

Go 方法是作用在接收者（receiver）上的一个函数，接收者是某种类型的变量。因此方法是一种特殊类型的函数。

接收者类型可以是（几乎）任何类型，不仅仅是结构体类型：任何类型都可以有方法，甚至可以是函数类型，可以是 int、bool、string 或数组的别名类型。但是接收者不能是一个接口类型，因为接口是一个抽象定义，但是方法却是具体实现；如果这样做会引发一个编译错误：`invalid receiver type…`。

最后接收者不能是一个指针类型，但是它可以是任何其他允许类型的指针。

定义方法的一般格式如下：

```go
func (recv receiver_type) methodName(parameter_list) (return_value_list) { ... }
```

在方法名之前，  func  关键字之后的括号中指定 receiver。

如果  recv  是 receiver 的实例，Method1 是它的方法名，那么方法调用遵循传统的  object.name  选择器符号：`recv.Method1()`。

如果  recv  是一个指针，Go 会自动解引用。

如果方法不需要使用  recv  的值，可以用 _ 替换它，比如：

```go
func (_ receiver_type) methodName(parameter_list) (return_value_list) { ... }
```

 recv  就像是面向对象语言中的  this  或  self  ，但是 Go 中并没有这两个关键字。

下面是一个结构体上的简单方法的例子：

```go
type TwoInts struct { 
    a int 
    b int 
} 
 
func main() { 
    two1 := new(TwoInts) 
    two1.a = 12 
    two1.b = 10 
 
    fmt.Printf("The sum is: %d\n", two1.AddThem()) 
    fmt.Printf("Add them to the param: %d\n", two1.AddToParam(20)) 
 
    two2 := TwoInts{3, 4} 
    fmt.Printf("The sum is: %d\n", two2.AddThem()) 
} 
 
func (tn *TwoInts) AddThem() int { 
    return tn.a + tn.b 
} 
 
func (tn *TwoInts) AddToParam(param int) int { 
    return tn.a + tn.b + param 
}
```

下面是非结构体类型上方法的例子：

```go
type IntVector []int 
 
func (v IntVector) Sum() (s int) { 
    for _, x := range v { 
        s += x 
    } 
    return 
} 
 
func main() { 
    fmt.Println(IntVector{1, 2, 3}.Sum()) // 输出是6 
}
```

#### 函数和方法的区别

函数将变量作为参数：Function1(recv)

方法在变量上被调用：recv.Method1()

在接收者是指针时，方法可以改变接收者的值（或状态），这点函数也可以做到（当参数作为指针传递，即通过引用调用时，
函数也可以改变参数的状态）。

****不要忘记 Method1 后边的括号 ()，否则会引发编译器错误：  method recv.Method1 is not an expression, must be**
called** 

接收者必须有一个显式的名字，这个名字必须在方法中被使用。

**receiver_type 叫做 （接收者）基本类型**，这个类型必须在和方法同样的包中被声明。

#### String() 方法

当定义了一个有很多方法的类型时，十之八九你会使用  String()  方法来定制类型的字符串形式的输出，换句话说：一种可阅读性和打印性的输出。如果类型定义了  String()  方法，它会被用在  fmt.Printf()  中生成默认的输出：等同于使用格式化描述符  %v  产生的输出。还有  fmt.Print()  和  fmt.Println()  也会自动使用 String()  方法。



### 接口

#### 接口是什么

Go 语言不是一种 “传统” 的面向对象编程语言：它里面没有类和继承的概念。

但是 Go 语言里有非常灵活的 接口 概念，通过它可以实现很多面向对象的特性。

接口定义了一组方法（方法集），但是这些方法不包含（实现）代码：它们没有被实现（它们是抽象的）。接口里也不能包含
变量。

通过如下格式定义接口：

```go
type Namer interface { 
    Method1(param_list) return_type 
    Method2(param_list) return_type 
    ... 
}
```

上面的  Namer  是一个 接口类型。

> （按照约定，只包含一个方法的）接口的名字由方法名加  [e]r  后缀组成，例如 Printer  、  Reader  、  Writer  、  Logger  、  Converter  等等。还有一些不常用的方式（当后缀er  不合适时），比如  Recoverable  ，此时接口名以  able  结尾，或者以  I  开头（像  .NET  或 Java  中那样）。

Go 语言中的接口都很简短，通常它们会包含 0 个、最多 3 个方法。

类型（比如结构体）实现接口方法集中的方法，每一个方法的实现说明了此方法是如何作用于该类型的：即实现接口，同时方法集也构成了该类型的接口。实现了  Namer  接口类型的变量可以赋值给  ai  （接收者值），此时方法表中的指针会指向被实现的接口方法。当然如果另一个类型（也实现了该接口）的变量被赋值给  ai  ，这二者（译者注：指针和方法实现）也会随之改变。

- **类型不需要显式声明它实现了某个接口：接口被隐式地实现。多个类型可以实现同一个接口。**
- **实现某个接口的类型（除了实现接口方法外）可以有其他的方法。**
- **一个类型可以实现多个接口。**
- **接口类型可以包含一个实例的引用， 该实例的类型实现了此接口（接口是动态类型）。**

例子：

```go
package main 
 
import "fmt" 
 
type Shaper interface { 
    Area() float32 
} 
 
type Square struct { 
    side float32 
} 
 
func (sq *Square) Area() float32 { 
    return sq.side * sq.side 
} 
 
type Rectangle struct { 
    length, width float32 
} 
 
func (r Rectangle) Area() float32 { 
    return r.length * r.width 
} 
 
func main() { 
 
    r := Rectangle{5, 3} // Area() of Rectangle needs a value 
    q := &Square{5}      // Area() of Square needs a pointer 
    // shapes := []Shaper{Shaper(r), Shaper(q)} 
    // or shorter 
    shapes := []Shaper{r, q} 
    fmt.Println("Looping through shapes for area ...") 
    for n, _ := range shapes { 
        fmt.Println("Shape details: ", shapes[n]) 
        fmt.Println("Area of this shape is: ", shapes[n].Area()) 
    } 
}
```

#### 接口嵌套接口

```go
type ReadWrite interface { 
    Read(b Buffer) bool 
    Write(b Buffer) bool 
} 
 
type Lock interface { 
    Lock() 
    Unlock() 
} 
 
type File interface { 
    ReadWrite 
    Lock 
    Close() 
}
```

#### 类型断言

一个接口类型的变量  varI  中可以包含任何类型的值，必须有一种方式来检测它的 动态 类型，即运行时在变量中存储的
值的实际类型。在执行过程中动态类型可能会有所不同，但是它总是可以分配给接口变量本身的类型。通常我们可以使用 类型
断言 来测试在某个时刻  varI  是否包含类型  T  的值：

```go
v := varI.(T)       // unchecked type assertion
```

varI 必须是一个接口变量，否则编译器会报错：  `invalid type assertion: varI.(T) (non-interface type (type ofvarI) on left) ` 。

类型断言可能是无效的，虽然编译器会尽力检查转换是否有效，但是它不可能预见所有的可能性。如果转换在程序运行时失败
会导致错误发生。更安全的方式是使用以下形式来进行类型断言：

```go
if v, ok := varI.(T); ok {  // checked type assertion 
    Process(v) 
    return 
} 
// varI is not of type T
```

如果转换合法，  v  是  varI  转换到类型  T  的值，  ok  会是  true  ；否则  v  是类型 T  的零值，  ok  是  false  ，也没有运行时错误发生。

#### 类型判断：type-switch

接口变量的类型也可以使用一种特殊形式的  switch  来检测：type-switch

```go
switch t := areaIntf.(type) { 
case *Square: 
    fmt.Printf("Type Square %T with value %v\n", t, t) 
case *Circle: 
    fmt.Printf("Type Circle %T with value %v\n", t, t) 
case nil: 
    fmt.Printf("nil value: nothing to check?\n") 
default: 
    fmt.Printf("Unexpected type %T\n", t) 
}
```

如果仅仅是测试变量的类型，不用它的值，那么就可以不需要赋值语句，比如：

```go
switch areaIntf.(type) { 
case *Square: 
    // TODO
case *Circle: 
    // TODO 
... 
default: 
    // TODO 
}
```



### 面向对象

> **Is Go an object-oriented language?** 
>
> **Yes and no.** Although Go has types and methods and allows an object-oriented style of programming, there is no type hierarchy. The concept of “interface” in Go provides a different approach that we believe is easy to use and in some ways more general.  Also, the lack of a type hierarchy makes “objects” in Go feel much more lightweight than in languages such as C++ or Java.
>
> —— [https://golang.org/doc/faq](https://golang.org/doc/faq)

#### 封装数据和行为

**结构体定义**

```go
type Employee struct {
  Id   string
  Name string
  Age  int
}
```

**实例例创建及初始化**

```go
func TestCreateEmployeeObj(t *testing.T) {
	e := Employee{"0", "Bob", 20}
	e1 := Employee{Name: "Mike", Age: 30}
	e2 := new(Employee) //返回指针
	e2.Id = "2"
	e2.Age = 22
	e2.Name = "Rose"
	t.Log(e)
	t.Log(e1)
	t.Log(e1.Id)
	t.Log(e2)
	t.Logf("e is %T", e)
	t.Logf("e2 is %T", e2)
}
```

**定义行为**

```go
//第⼀一种定义⽅方式在实例例对应⽅方法被调⽤用时，实例例的成员会进⾏行行值复制
func (e Employee) String() string {
	fmt.Printf("Address is %x\n", unsafe.Pointer(&e.Name))
	return fmt.Sprintf("ID:%s-Name:%s-Age:%d", e.Id, e.Name, e.Age)
}

//通常情况下为了了避免内存拷⻉贝我们使⽤用第⼆二种定义⽅方式
//func (e *Employee) String() string {
//	fmt.Printf("Address is %x", unsafe.Pointer(&e.Name))
//	return fmt.Sprintf("ID:%s/Name:%s/Age:%d", e.Id, e.Name, e.Age)
//}
```

**自定义类型**

1. `type IntConvertionFn func(n int) int`
2. `type MyPoint int`

#### 接口与依赖

Duck Type 式接口实现

1. 接口为非入侵性，实现不不依赖于借⼝口定义
2. 所以接⼝口的定义可以包含在接⼝口使⽤用者包内

```go
// 接口
type Programmer interface {
	WriteHelloWorld() string
}

// 结构体 （接口实现）
type GoProgrammer struct {
}

// 接口实现方法
func (g *GoProgrammer) WriteHelloWorld() string {
	return "fmt.Println(\"Hello World\")"
}

// 测试
func TestClient(t *testing.T) {
	var p Programmer
	p = new(GoProgrammer)
	t.Log(p.WriteHelloWorld())
}
```

接口变量：

![image-20220209090909477](https://cos.duktig.cn/typora/202202090909748.png)

#### 扩展与复用

Go 不支持继承，但可以通过复合的方式来复用

```go
type Pet struct {
}

func (p *Pet) Speak() {
	fmt.Print("...")
}

func (p *Pet) SpeakTo(host string) {
	p.Speak()
	fmt.Println(" ", host)
}

// 匿名复用
type Dog struct {
	Pet
}

// 不能重载 相当于父类的方法
func (d *Dog) Speak() {
	fmt.Print("Wang!")
}

func TestDog(t *testing.T) {
	dog := new(Dog)

	dog.SpeakTo("Chao")
}
```

**匿名类型嵌入**

它不是继承，如果我们把“内部 struct ”看作父类，把“外部 struct”  看作子类，会发现如下问题：

1. 不支持子类替换
2. 子类并不是真正继承了父类的方法，父类的定义的方法无法访问子类的数据和方法

#### 多态

![image-20220209093724048](C:\Users\rsw\AppData\Roaming\Typora\typora-user-images\image-20220209093724048.png)

```go
type Code string
type Programmer interface {
	WriteHelloWorld() Code
}

type GoProgrammer struct {
}

func (p *GoProgrammer) WriteHelloWorld() Code {
	return "fmt.Println(\"Hello World!\")"
}

type JavaProgrammer struct {
}

func (p *JavaProgrammer) WriteHelloWorld() Code {
	return "System.out.Println(\"Hello World!\")"
}

func writeFirstProgram(p Programmer) {
	fmt.Printf("%T %v\n", p, p.WriteHelloWorld())
}

func TestPolymorphism(t *testing.T) {
	goProg := &GoProgrammer{}
	javaProg := new(JavaProgrammer)
	writeFirstProgram(goProg)
	writeFirstProgram(javaProg)
}
```



### 空接口与断言

1. 空接⼝口可以表示任何类型
2. 通过断⾔言来将空接⼝口转换为制定类型  `v, ok := p.(int) //ok=true 时为转换成功`

```go
func DoSomething(p interface{}) {
	// if i, ok := p.(int); ok {
	// 	fmt.Println("Integer", i)
	// 	return
	// }
	// if s, ok := p.(string); ok {
	// 	fmt.Println("stirng", s)
	// 	return
	// }
	// fmt.Println("Unknow Type")
	switch v := p.(type) {
	case int:
		fmt.Println("Integer", v)
	case string:
		fmt.Println("String", v)
	default:
		fmt.Println("Unknow Type")
	}
}

func TestEmptyInterfaceAssertion(t *testing.T) {
	DoSomething(10)
	DoSomething("10")
}
```

**Go 接⼝口最佳实践**

![image-20220209095239609](https://cos.duktig.cn/typora/202202090952314.png)

### 读写数据

#### 用户输入输出

从键盘和标准输入  os.Stdin  读取输入，最简单的办法是使用  fmt 包提供的 Scan 和 Sscan 开头的函数。

除了 fmt 和 os 包，我们还需要用到 buﬁo 包来处理缓冲的输入和输出。

```go
inputReader := bufio.NewReader(os.Stdin)
```

#### 读写文件

在 Go 语言中，文件使用指向  `os.File`  类型的指针来表示的，也叫做文件句柄。标准输入 `os.Stdin`  和标准输出  `os.Stdout`  ，他们的类型都是  `*os.File`  。









### 异常处理

1. 没有异常机制
2. error 类型实现了了 error 接⼝口
3. 可以通过 errors.New 来快速创建错误实例例

```go
type error interface {
    Error() string
}
errors.New("n must be in the range [0,100]")
```

定义不不同的错误变量量，以便便于判断错误类型:

```go
var LessThanTwoError error = errors.New("n must be greater than 2")
var GreaterThanHundredError error = errors.New("n must be less than 100")

func TestGetFibonacci(t *testing.T) {
    var list []int
    list, err := GetFibonacci(-10)
    if err == LessThanTwoError {
        t.Error("Need a larger number")
    }
    if err == GreaterThanHundredError {
        t.Error("Need a larger number")
    }
    …
}
```



#### panic

- panic ⽤用于不不可以恢复的错误
- panic 退出前会执⾏行行 defer 指定的内容

**panic vs. os.Exit** 

- os.Exit 退出时不不会调⽤用 defer 指定的函数
- os.Exit 退出时不不输出当前调⽤用栈信息

#### recover

```go
defer func() {
if err := recover(); err != nil {
     //恢复错误
	}
}()
```



### 依赖管理

**Go 未解决的依赖问题**

1. 同一环境下，不同项目使用同一包的不同版本
2. 无法管理对包的特定版本的依赖

**vendor 路径**

随着 Go 1.5 release 版本的发布，vendor ⽬目录被添加到除了 GOPATH 和GOROOT 之外的依赖目录查找的解决方案。在 Go 1.6 之前，你需要手动的设置环境变量量查找依赖包路路径的解决方案如下：

1. 当前包下的 vendor 目录
2. 向上级目录查找，直到找到 src 下的 vendor 目录
3. 在 GOPATH 下⾯面查找依赖包
4. 在 GOROOT 目录下查找

### 并发机制

#### 携程

Thead vs. Groutine
1. 创建时默认的 stack 的⼤大⼩小 
  1. JDK5 以后 Java Thread stack 默认为1M
  2. Groutine 的 Stack 初始化⼤大⼩小为2K
2. 和 KSE （Kernel Space Entity) 的对应关系 
  1. Java Thread 是 1:1
  2. Groutine 是 M:N

```go
func TestGroutine(t *testing.T) {
	for i := 0; i < 10; i++ {
		go func(i int) {
			//time.Sleep(time.Second * 1)
			fmt.Println(i)
		}(i)
	}
	time.Sleep(time.Millisecond * 50)
}
```



#### 共享内存并发机制

```go
// 携程 共享内存 不安全
func TestCounter(t *testing.T) {

	counter := 0
	for i := 0; i < 5000; i++ {
		go func() {
			counter++
		}()
	}
	time.Sleep(1 * time.Second)
	t.Logf("counter = %d", counter)
}

// 使用锁 使携程安全
func TestCounterThreadSafe(t *testing.T) {
	var mut sync.Mutex
	counter := 0
	for i := 0; i < 5000; i++ {
		go func() {
			defer func() {
				mut.Unlock()
			}()
			mut.Lock()
			counter++
		}()
	}
	time.Sleep(1 * time.Second)
	t.Logf("counter = %d", counter)
}

// 等待唤醒 机制
func TestCounterWaitGroup(t *testing.T) {
	var mut sync.Mutex
	var wg sync.WaitGroup
	counter := 0
	for i := 0; i < 5000; i++ {
		wg.Add(1)
		go func() {
			defer func() {
				mut.Unlock()
			}()
			mut.Lock()
			counter++
			wg.Done()
		}()
	}
	wg.Wait()
	t.Logf("counter = %d", counter)
}
```

#### CSP 并发机制

##### Channel



##### 多路路选择和超时控制

select

多渠道的选择：

```go
select {
case ret := <-retCh1:
t.Logf("result %s", ret)
case ret :=<-retCh2:
              t.Logf("result %s", ret)
   default:
t.Error(“No one returned”)
}
```

超时控制:

```go
select {
case ret := <-retCh:
t.Logf("result %s", ret)
case <-time.After(time.Second * 1):
t.Error("time out")
}
```

##### Channel关闭

- 向关闭的 channel 发送数据，会导致 panic
- v, ok <-ch; ok 为 bool 值，true 表示正常接受，false 表示通道关闭
- 所有的 channel 接收者都会在 channel 关闭时，⽴立刻从阻塞等待中返回且上述 ok 值为 false。这个⼴广播机制常被利利⽤用，进⾏行行向多个订阅者同时发送信号。如：退出信号。

##### 取消

获取取消通知

```go
func isCancelled(cancelChan chan struct{}) bool {
    select {
        case <-cancelChan:
        return true
        default:
        return false
    }
}
```

发送取消消息

```go
func cancel_1(cancelChan chan struct{}) {
    cancelChan <- struct{}{}
}
```

通过关闭 Channel 取消

```GO
func cancel_2(cancelChan chan struct{}) {
    close(cancelChan)
}
```

#### Context

- 根 Context：通过 context.Background () 创建
- 子 Context：context.WithCancel(parentContext) 创建
- ctx, cancel := context.WithCancel(context.Background())
- 当前 Context 被取消时，基于他的⼦子 context 都会被取消
- 接收取消通知 <-ctx.Done()



### 常见并发任务

#### 仅执行一次

单例例模式 （懒汉式，线程安全）

```go
type Singleton struct {
	data string
}

var singleInstance *Singleton
var once sync.Once

// 单例模式实现
func GetSingletonObj() *Singleton {
	// 只执行一次
	once.Do(func() {
		fmt.Println("Create Obj")
		singleInstance = new(Singleton)
	})
	return singleInstance
}

func TestGetSingletonObj(t *testing.T) {
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			obj := GetSingletonObj()
			fmt.Printf("%X\n", unsafe.Pointer(obj))
			wg.Done()
		}()
	}
	wg.Wait()
}
```

#### 仅需任意任务完成

> 场景：比如多任务同时向百度、谷歌、搜狗等进行搜索，只要有一个搜索到想要的结果即可返回。

```go
// 运行任务
func runTask(id int) string {
   time.Sleep(10 * time.Millisecond)
   return fmt.Sprintf("The result is from %d", id)
}

// 第一个任务执行完后 即返回
func FirstResponse() string {
   numOfRunner := 10
   // buffer channel
   ch := make(chan string, numOfRunner)
   for i := 0; i < numOfRunner; i++ {
      go func(i int) {
         ret := runTask(i)
         ch <- ret
      }(i)
   }
   // 通道有消息 直接返回
   return <-ch
}

func TestFirstResponse(t *testing.T) {
   t.Log("Before:", runtime.NumGoroutine())
   t.Log(FirstResponse())
   time.Sleep(time.Second * 1)
   t.Log("After:", runtime.NumGoroutine())
}
```

#### 必需所有任务完成

```go
// 所有任务完成 再返回
func AllResponse() string {
   numOfRunner := 10
   ch := make(chan string, numOfRunner)
   for i := 0; i < numOfRunner; i++ {
      go func(i int) {
         ret := runTask(i)
         ch <- ret
      }(i)
   }
   finalRet := ""
   for j := 0; j < numOfRunner; j++ {
      finalRet += <-ch + "\n"
   }
   return finalRet
}

func TestAllResponse(t *testing.T) {
   t.Log("Before:", runtime.NumGoroutine())
   t.Log(AllResponse())
   time.Sleep(time.Second * 1)
   t.Log("After:", runtime.NumGoroutine())
}
```

### 对象池

使用buﬀered channel实现对象池



### sync.Pool 对象缓存



### 测试

#### 单元测试

- Fail, Error: 该测试失败，该测试继续，其他测试继续执行
- FailNow, Fatal: 该测试失败，该测试中止，其他测试继续执行

代码覆盖率 ` go test -v - cover`

断言:  [https://github.com/stretchr/testify](https://github.com/stretchr/testify)

#### Benchmark

```go
func BenchmarkConcatStringByAdd(b *testing.B) {
//与性能测试⽆无关的代码
b.ResetTimer()
for i := 0; i < b.N; i++ {
//测试代码
}
b.StopTimer()
    //与性能测试⽆无关的代码
}
```

`go test -bench=. -benchmem`

- -bench=<相关benchmark测试>
- Windows 下使⽤用 go test 命令⾏行行时，-bench=.应写为-bench="."

### 反射

- reﬂect.TypeOf 返回类型 (reﬂect.Type)
- reﬂect.ValueOf 返回值 (reﬂect.Value)
- 可以从 reﬂect.Value 获得类型
- 通过 kind 的来判断类型

- 按名字访问结构的成员：`reflect.ValueOf(*e).FieldByName("Name")`

- 按名字访问结构的⽅方法：`reflect.ValueOf(e).MethodByName("UpdateAge").Call([]reflect.Value{reflect.ValueOf(1)})`

### unsafe

“不不安全”⾏行行为的危险性

```go
i := 10
f := *(*float64)(unsafe.Pointer(&i))
```



## Go进阶

### Json解析

EasyJSON   采⽤用代码⽣生成⽽而⾮非反射

安装

go get -u github.com/mailru/easyjson/...

使用

easyjson -all <结构定义>.go

### 构建 RestFul 服务

http

```go
func Hello(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	fmt.Fprintf(w, "hello, %s!\n", ps.ByName("name"))
}

func main() {
	router := httprouter.New()
	router.GET("/", Index)
	router.GET("/hello/:name", Hello)
	log.Fatal(http.ListenAndServe(":8080", router))
}
```

使用第三方

[https://github.com/julienschmidt/httprouter](https://github.com/julienschmidt/httprouter)

```go
func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
   fmt.Fprint(w, "Welcome!\n")
}

func Hello(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
   fmt.Fprintf(w, "hello, %s!\n", ps.ByName("name"))
}

func main() {
   router := httprouter.New()
   router.GET("/", Index)
   router.GET("/hello/:name", Hello)

   log.Fatal(http.ListenAndServe(":8080", router))
}
```













### 注意事项

Go语言不允许使用无用的局部变量（local	variables），因为这会导致编译错误。

Go语言中这种情况的解决方法是用	**空标识符**	（blank	identifier），即	`_`	（也就是下划线）。空标识符可用于任何语法需要变量名但程序逻辑不需要的时候,	例如,	在循环里，丢弃不需要的循环索引,	保留元素值。

+=	连接原字符串	原来的内容已经不再使用，将在适当时机对它进行垃圾回收。如果连接涉及的数据量很大，这种方式代价高昂。一种简单且高效的解决方案是使用	strings	包的	`Join`	函数。





