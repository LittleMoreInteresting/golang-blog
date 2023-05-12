+++
title = "基础篇（一）：包、变量和方法"
description = "基础篇（一）：包、变量和方法；学习任何Go程序的基本组成部分。"
weight = 1
url = "/tour/01.html"
+++
## 包（ package）
每个Go项目都是由程序包组成的。
程序在main包中运行。
```shell
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	fmt.Println("My favorite number is", rand.Intn(10))
}

```
此程序使用导入的“fmt”和“math/rand”包。
按照惯例，包名称为导入路径的最后一个元素。例如，“math/rand”包含以rand为包名的文件。
### import
此代码将导入分组到一个带括号的的import语句中。
```go
package main

import (
	"fmt"
	"math"
)

func main() {
	fmt.Printf("Now you have %g problems.\n", math.Sqrt(7))
}

```
您还可以编写多个import语句，如：
```go
import "fmt"
import "math"
```
但是，建议使用带括号的import语句。
### 导出名称
在 Go 中，如果名称以大写字母开头，则会导出该名称。例如，Pizza，Pi 是是从math包中导出的名称。
pizza并且pi不要以大写字母开头，因此它们不会被导出。
导入包时，您只能引用其导出的名称。任何“未导出”的名称都无法从包外部访问。
运行代码。请注意错误消息。
要修复错误，请重命名math.pi为math.Pi并重试。
```go
package main

import (
	"fmt"
	"math"
)

func main() {
	fmt.Println(math.pi)
}

```
## 函数/方法 (Functions)
### Functions
函数可以接受零个或多个参数。
在本例中，add接收两个int类型的参数。
> 请注意，该类型位于变量名之后。

```go
package main

import "fmt"

func add(x int, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
}
```
当两个或多个连续的命名函数参数共享一个类型时，可以从除最后一个之外的所有参数中省略该类型。在这个例子中，可以把
x int, y int 简写为 x, y int 
### 多返回值 Multiple results 
函数可以返回任意数量的结果。函数可以返回任意数量的结果。
```go
package main

import "fmt"

func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```
### 具名返回值 Named return values 
Go的返回值可以命名。这种写法将视为在函数顶部定义的变量。这些变量用于记录返回值。不带参数的return语句返回命名的返回值。这种方式适合比较短的函数；如果是函数体比较长的函数这种方式可读性会比较差
```go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}

```

## 变量（Variables）
var语句声明了一个变量列表；在函数参数列表中，类型是最后一个。
var语句可以是包级别的，也可以是函数级别的。例子中分别展示了这两种情况。
```go
package main

import "fmt"

var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}
```
### 定义带初始值的变量
var 声明每个变量包含初始值的变量。
如果存在初始值，则可以省略类型；该变量将采用初始值设定项的类型。
```go
package main

import "fmt"

var i, j int = 1, 2

func main() {
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}
```
### 短变量声明

在函数内部，可以使用 := 短赋值语句来代替隐式类型的var声明。在函数之外，每条语句都以关键字（var、func等）开头，因此:=构造不可用。
```go
package main

import "fmt"

func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"

	fmt.Println(i, j, k, c, python, java)
}

```
### 基本类型
Go语言的基本类型有
```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
```
该示例显示了几种类型的变量，还显示了变量声明可以被“分解”到块中，就像import语句一样
```go
package main

import (
	"fmt"
	"math/cmplx"
)

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}

```
int、uint和uintptr类型在32位系统上通常为32位宽，在64位系统上为64位宽。当您需要一个整数值时，您应该使用int，除非您有特定的理由使用大小或无符号整数类型。

### 零值

在没有显式初始值的情况下声明的变量将被赋予零值。
各类型零值:

- 数字类型 0,
- 布尔类型 false 
- string类型 "" (空字符串) .
```go
package main

import "fmt"

func main() {
	var i int
	var f float64
	var b bool
	var s string
	fmt.Printf("%v %v %v %q\n", i, f, b, s)
}

```
### 类型转换

表达式T(v) 将值v转换为类型T。
一些数字转换
```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```
或者，更简单地说：
```go
i := 42
f := float64(i)
u := uint(f)
```
与C不同，Go语言不同类型的项之间的赋值需要显式转换。尝试删除示例中的float64或uint转换，看看会发生什么。
```go
package main

import (
	"fmt"
	"math"
)

func main() {
	var x, y int = 3, 4
	var f float64 = math.Sqrt(float64(x*x + y*y))
	var z uint = uint(f)
	fmt.Println(x, y, z)
}
```
### 类型推断
当声明变量而不指定显式类型时（通过使用 := 语法或var =表达式语法），变量的类型是从右侧的值推断出来的。
当声明的右侧被输入时，新变量是相同类型的
```go
var i int
j := i // j 是 int 类型
```
但是，当右侧包含非类型化的数字常量时，新变量可能是int、float64或complex128，具体取决于常量的精度：
```go
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```
```go
package main

import "fmt"

func main() {
	v := 42 // change me!
	fmt.Printf("v is of type %T\n", v)
}

```
尝试更改示例代码中v的初始值，并观察其类型是如何受到影响的。

### 常量

常量是像变量一样声明的，但使用const关键字。常量可以是字符、字符串、布尔值或数值。不能使用 := 语法声明常量。
```go
package main

import "fmt"

const Pi = 3.14

func main() {
	const World = "世界"
	fmt.Println("Hello", World)
	fmt.Println("Happy", Pi, "Day")

	const Truth = true
	fmt.Println("Go rules?", Truth)
}
```
### 数值常量
数值常量是高精度的值。非类型化的常量采用其上下文所需的类型。
```go
package main

import "fmt"

const (
	// 通过向左移动1位100位来创建一个巨大的数字。
	// 换句话说，二进制数是1，后面跟着100个零。
	Big = 1 << 100
	// 再次向右移动99位，所以我们最终得到1<<1或2。
	Small = Big >> 99
)

func needInt(x int) int { return x*10 + 1 }
func needFloat(x float64) float64 {
	return x * 0.1
}

func main() {
	fmt.Println(needInt(Small))
	fmt.Println(needFloat(Small))
	fmt.Println(needFloat(Big))
}
```
也可以尝试打印needInt（Big）。（int最多可以存储64位整数，有时可以存储更少的整数。）
