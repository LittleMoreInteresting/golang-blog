+++
title = "4、方法与接口"
description = "方法与接口;Go没有类。但是，您可以在类型上定义方法。"
weight = 4
url = "/tour/04.html"
+++

## Methods
Go没有类。但是，您可以在类型上定义方法。
方法是一个具有特殊接收器参数的函数。
接收器出现在自己的参数列表中，位于func关键字和方法名称之间。
```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}

```
在这个例子中，Abs方法有一个名为v的Vertex类型的接收器。
请记住：方法只是一个带有接收器参数的函数。下面的Abs是作为一个普通函数编写的，函数本身没有变化。
```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(Abs(v))
}

```
您也可以在非结构类型上声明一个方法。
```go
package main

import (
	"fmt"
	"math"
)

type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

func main() {
	f := MyFloat(-math.Sqrt2)
	fmt.Println(f.Abs())
}

```
在这个例子中，我们看到一个带有Abs方法的数字类型MyFloat。只能声明具有接收器的方法，该接收器的类型与该方法在同一个包中定义。不能使用类型在另一个包（包括int等内置类型）中定义的接收器来声明方法。
### 指针接收器
可以使用指针接收器声明方法。这意味着接收器类型具有某些类型T的指针*T。（此外，T本身不能是*int之类的指针。）
例如，此处的“Scale”方法是在“*顶点”上定义的。
```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}

```
带有指针接收器的方法可以修改接收器指向的值（就像Scale在这里所做的那样）。由于方法通常需要修改其接收器，因此指针接收器比值接收器更常见。
尝试从第16行Scale函数的声明中删除*，并观察程序的行为是如何变化的。
对于值接收器，“Scale”方法对原始“Vertex ”值的副本进行操作。（这与任何其他函数参数的行为相同。）Scale方法必须有一个指针接收器才能更改主函数中声明的Vertex值。
### 指针和函数
在这里，我们看到Abs和Scale方法被重写为函数。
```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func Scale(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	Scale(&v, 10)
	fmt.Println(Abs(v))
}

```
再次尝试从第16行删除*。你能看到为什么行为会改变吗？为了编译示例，您还需要更改哪些内容？
### 方法和指针间接寻址
比较前两个程序，您可能会注意到带有指针参数的函数必须使用指针：
```go
var v Vertex
ScaleFunc(v, 5)  // Compile error!
ScaleFunc(&v, 5) // OK
```
而具有指针接收器的方法在被调用时采用值或指针作为接收器：
```go
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK
```
对于v.Scale（5）语句，即使v是一个值而不是指针，也会自动调用带有指针接收器的方法。也就是说，为了方便起见，Go将语句v.Scale（5）解释为（&v）.Scale（5），因为Scale方法有一个指针接收器。
```go
package main

import "fmt"

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func ScaleFunc(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(2)
	ScaleFunc(&v, 10)

	p := &Vertex{4, 3}
	p.Scale(3)
	ScaleFunc(p, 8)

	fmt.Println(v, p)
}

```
反过来，采用值参数的函数必须采用该特定类型的值：
```go
var v Vertex
fmt.Println(AbsFunc(v))  // OK
fmt.Println(AbsFunc(&v)) // Compile error!
```
而具有值接收器的方法在被调用时可以采用值或指针作为接收器：
```go
var v Vertex
fmt.Println(v.Abs()) // OK
p := &v
fmt.Println(p.Abs()) // OK
```
在这种情况下，方法调用p.Abs（）被解释为（*p）.Abs（）。
```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func AbsFunc(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
	fmt.Println(AbsFunc(v))

	p := &Vertex{4, 3}
	fmt.Println(p.Abs())
	fmt.Println(AbsFunc(*p))
}

```
### 选择值或指针接收器
使用指针接收器有两个原因。第一个是使该方法可以修改其接收器指向的值。第二个是在方法调用时避免复制。如果接收器是一个大型结构体，这可能会更高效。
```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := &Vertex{3, 4}
	fmt.Printf("Before scaling: %+v, Abs: %v\n", v, v.Abs())
	v.Scale(5)
	fmt.Printf("After scaling: %+v, Abs: %v\n", v, v.Abs())
}

```
在本例中，Scale和Abs都是接收器类型为*Vertex的方法，尽管Abs方法不需要修改其接收器。
通常，给定类型上的所有方法都应该具有值或指针接收器，但不能同时具有这两者。
## Interfaces
接口类型被定义为一组方法签名。
接口类型的值可以包含实现这些方法的任何值。
```go
package main

import (
	"fmt"
	"math"
)

type Abser interface {
	Abs() float64
}

func main() {
	var a Abser
	f := MyFloat(-math.Sqrt2)
	v := Vertex{3, 4}

	a = f  // a MyFloat implements Abser
	a = &v // a *Vertex implements Abser

	// In the following line, v is a Vertex (not *Vertex)
	// and does NOT implement Abser.
	a = v

	fmt.Println(a.Abs())
}

type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

```
注意：第22行的示例代码中有一个错误。Vertex（值类型）不实现Abser，因为Abs方法仅在*Vertex（指针类型）上定义。
### 接口是隐式实现的
类型通过实现其方法来实现接口。没有明确的意向声明，也没有“implements”关键字。隐式接口将接口的定义与其实现解耦，然后接口可以在没有预先安排的情况下出现在任何包中。
```go
package main

import "fmt"

type I interface {
	M()
}

type T struct {
	S string
}

// This method means type T implements the interface I,
// but we don't need to explicitly declare that it does so.
func (t T) M() {
	fmt.Println(t.S)
}

func main() {
	var i I = T{"hello"}
	i.M()
}

```
### 接口值
接口值可以被认为是一个值和一个具体类型的元组 (value, type)
接口值包含特定基础具体类型的值。对接口值调用方法会在其基础类型上执行相同名称的方法。
```go

package main

import (
	"fmt"
	"math"
)

type I interface {
	M()
}

type T struct {
	S string
}

func (t *T) M() {
	fmt.Println(t.S)
}

type F float64

func (f F) M() {
	fmt.Println(f)
}

func main() {
	var i I

	i = &T{"Hello"}
	describe(i)
	i.M()

	i = F(math.Pi)
	describe(i)
	i.M()
}

func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}

```
### 基本值为nil的接口值
如果接口内部的具体值为nil，那么将使用nil接收器调用该方法。在某些语言中，这会触发null指针异常，但在Go中，通常会编写一些方法来优雅地处理使用nil接收器调用的情况（如本例中的方法M）
```go
package main

import "fmt"

type I interface {
	M()
}

type T struct {
	S string
}

func (t *T) M() {
	if t == nil {
		fmt.Println("<nil>")
		return
	}
	fmt.Println(t.S)
}

func main() {
	var i I

	var t *T
	i = t
	describe(i)
	i.M()

	i = &T{"hello"}
	describe(i)
	i.M()
}

func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}

```
请注意，持有nil具体值的接口值本身就是非nil。

Nil接口值
nil接口值既不包含值，也不包含具体类型。在nil接口上调用方法是一个运行时错误，因为接口元组中没有指示要调用哪个具体方法的类型。
```go
package main

import "fmt"

type I interface {
	M()
}

func main() {
	var i I
	describe(i)
	i.M()
}

func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}

```
### 空接口
指定零方法的接口类型称为空接口：interface{}
空接口可以包含任何类型的值。（每个类型至少实现零个方法。）
空接口在程序处理未知类型值的时候使用。例如，fmt.Print接受任意数量的接口｛｝类型的参数。
```go
package main

import "fmt"

func main() {
	var i interface{}
	describe(i)

	i = 42
	describe(i)

	i = "hello"
	describe(i)
}

func describe(i interface{}) {
	fmt.Printf("(%v, %T)\n", i, i)
}

```
### 类型断言
类型断言提供了访问接口值的底层具体值的方法。
```go
t := i.(T)
```
该语句断言接口值i包含具体类型T，并将底层T值分配给变量t。如果不是T类型，那么这个语句将引发panic。
为了测试接口值是否包含特定类型，类型断言可以返回两个值：基础值和报告断言是否成功的布尔值。
```go
t, ok := i.(T)
```
如果是T类型，那么t被赋值值，ok将为true
如果不是，ok将为false，t将是类型T的零值，并且不会发生panic。
注意这个语法和从map的语法之间的相似性。
```go
package main

import "fmt"

func main() {
	var i interface{} = "hello"

	s := i.(string)
	fmt.Println(s)

	s, ok := i.(string)
	fmt.Println(s, ok)

	f, ok := i.(float64)
	fmt.Println(f, ok)

	f = i.(float64) // panic
	fmt.Println(f)
}

```
### 类型转换
类型转换是一种允许多个类型断言串联的构造。
类型转换类似于常规switch语句，但类型转换中的case指定类型（而不是值），并将这些值与给定接口值所持有的值的类型进行比较。
```go
switch v := i.(type) {
case T:
    // here v has type T
case S:
    // here v has type S
default:
    // no match; here v has the same type as i
}
```
类型转换中的声明与类型断言i具有相同的语法 i.(T)，但特定类型T被关键字 type 所取代。
```go
package main

import "fmt"

func do(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}

func main() {
	do(21)
	do("hello")
	do(true)
}

```

此switch语句测试接口值i是否持有T或S类型的值。在T和S的每种情况下，变量v将分别为T或S型，并持有i持有的值。默认情况下（不匹配），变量v与i具有相同的接口类型和值。
### Stringer 接口
fmt包定义的Stringer是最普遍的接口之一。
```go
type Stringer interface {
    String() string
}
```
Stringer是一种可以将自己描述为字符串的类型。fmt包（以及许多其他包）查找此接口以打印值。
```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
	a := Person{"Arthur Dent", 42}
	z := Person{"Zaphod Beeblebrox", 9001}
	fmt.Println(a, z)
}

```
### 练习：Stringer
使IPAddr类型实现fmt.Stringer，以将地址打印为点四元组。例如，IPAddr｛1，2，3，4｝应打印为“1.2.3.4”。
_代码仅供参考_
```go
package main

import "fmt"

type IPAddr [4]byte

// TODO: Add a "String() string" method to IPAddr.
func (ip IPAddr) String() string {
	return fmt.Sprintf("%d.%d.%d.%d", ip[0], ip[1], ip[2], ip[3])
}
func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}
```
### Errors 接口
Go程序用错误值表示错误状态。错误类型是一个类似于fmt.Stringer的内置接口。：
```go
type error interface {
    Error() string
}
```
（与fmt.Stringer一样，fmt包在打印值时会查找error接口。）
函数通常返回一个错误值，调用代码应该通过测试错误是否等于零来处理错误。
```go
i, err := strconv.Atoi("42")
if err != nil {
    fmt.Printf("couldn't convert number: %v\n", err)
    return
}
fmt.Println("Converted integer:", i)
```
nil 表示成功；非 nil 表示失败。
```go
package main

import (
	"fmt"
	"time"
)

type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s",
		e.When, e.What)
}

func run() error {
	return &MyError{
		time.Now(),
		"it didn't work",
	}
}

func main() {
	if err := run(); err != nil {
		fmt.Println(err)
	}
}

```
### 练习：error
复制前面练习中的Sqrt函数，并对其进行修改以返回错误值。当给定负数时，Sqrt应该返回一个非零错误值，因为它不支持复数。创建新类型 
```go
type ErrNegativeSqrt float64
```
 实现error 接口
```go
func (e ErrNegativeSqrt) Error() string
```
使得ErrNegativeSqrt（-2）.Error（）返回"cannot Sqrt negative number: -2".。
```go
package main

import (
	"fmt"
	"math"
)

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
	return fmt.Sprintf("cannot Sqrt negative number: %v", float64(e))
}

func Sqrt(x float64) (float64, error) {
	if x < 0 {
		return 0, ErrNegativeSqrt(x)
	}
	var z = 1.0
	var old float64
	for math.Abs(old-z) >= 0.000001 {
		old = z
		z -= (z*z - x) / (2 * z)
	}
	return z, nil
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}

```

### Reader 接口
io包指定io.Reader接口，该接口表示数据流的读取端。Go标准库包含该接口的许多实现，包括文件、网络连接、压缩器、密码等。io.Reader接口有一个Read方法：
```go
func (T) Read(b []byte) (n int, err error)
```
Read用数据填充给定的字节片，并返回填充的字节数和错误值。当流结束时，它返回一个io.EOF错误。
示例代码创建一个strings.Reader，并一次消耗其输出的8个字节。
```go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	r := strings.NewReader("Hello, Reader!")

	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}

```
### 练习：Reader
实现一个Reader类型，该类型发出ASCII字符“a”的无限流。
_参考_
```go
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

// TODO: Add a Read([]byte) (int, error) method to MyReader.
func (mr MyReader) Read(b []byte) (int, error) {
	for i := range b {
		b[i] = 'A'
	}
	return len(b), nil
}
func main() {
	reader.Validate(MyReader{})
}

```
### 练习：rot13Reader
一种常见的模式是一个io.Reader，它包装另一个io.Leader，以某种方式修改流。例如，gzip.NewReader函数接受一个io.Reader（压缩数据流），并返回一个*gzip.Reader，该函数还实现了io.Readers（解压缩数据流）。实现一个rot13Reader，它实现io.Reader并从io.Readers读取，通过将rot13替换密码应用于所有字母字符来修改流。rot13Reader类型是为您提供的。通过实现它的Read方法，使它成为一个io.Reader。
```go
package main

import (
	"io"
	"os"
	"strings"
)

type rot13Reader struct {
	r io.Reader
}

func (rr *rot13Reader) Read(p []byte) (n int, err error) {

	for {
		b := make([]byte, 1024, 2048)
		read, err1 := rr.r.Read(b)
		if err1 == io.EOF {
			break
		}
		for i, b2 := range b[:read] {
			p[i] = rot13byte(b2)
		}
		n += read
	}
	return n, nil
}
func rot13byte(b byte) byte {
	s := rune(b)
	if s >= 'a' && s <= 'm' || s >= 'A' && s <= 'M' {
		b += 13
	}
	if s >= 'n' && s <= 'z' || s >= 'N' && s <= 'Z' {
		b -= 13
	}

	return b
}
func main() {
	s := strings.NewReader("Lbh penpxrq gur pbqr!")
	r := rot13Reader{s}
	io.Copy(os.Stdout, &r)
}

```
### Images
image 包定义 image 接口：
```go
package image

type Image interface {
    ColorModel() color.Model
    Bounds() Rectangle
    At(x, y int) color.Color
}
```
注意：Bounds方法的Rectangle返回值实际上是一个图像。Rectangle，因为声明在包图像中。color.color和color.Model类型也是接口，但我们将通过使用预定义的实现color.RGBA和color.RGBAModel来忽略这一点。这些接口和类型由 image/color 包指定
```go
package main

import (
	"fmt"
	"image"
)

func main() {
	m := image.NewRGBA(image.Rect(0, 0, 100, 100))
	fmt.Println(m.Bounds())
	fmt.Println(m.At(0, 0).RGBA())
}

```
### 练习：image
还记得你之前写的图片生成器吗？让我们再写一个，但这次它将返回image.image的实现，而不是数据切片。
定义您自己的图像类型，实现必要的方法，然后调用pic.ShowImage。
Bounds应该返回一个 image.Rectangle，类似于图像。矩形（0，0，w，h）。
ColorModel应返回color.RGBAModel。
At应该返回一种颜色；最后一个图片生成器中的值v对应于颜色。RGBA{v，v，255，255}。
```go
package main

import (
	"image"
	"image/color"

	"golang.org/x/tour/pic"
)

type Image struct{}

func (i Image) ColorModel() color.Model {
	return color.RGBAModel
}

func (i Image) Bounds() image.Rectangle {
	return image.Rect(0, 0, 200, 200)
}

func (i Image) At(x, y int) color.Color {
	return color.RGBA{uint8(x % 255), uint8(y % 255), 255, 255}
}

func main() {
	m := Image{}
	pic.ShowImage(m)
}

```
![image.png](/img/03/image_methods_0004.png)

