+++
title = "基础篇（三）：更多类型: struct, slices, 和 map"
description = "基础篇（三）：更多类型: struct, slices, 和 map"
weight = 3
url = "/tour/03.html"
+++
## 指针
Go提供了指针类型。指针保存的是一个值的内存地址。
类型*T是指向T值的指针。它的零值为nil。
```go
var p *int
```
&运算符生成指向其操作数的指针。
```go
i := 42
p = &i
```
*运算符表示指针的基本值。
```go
fmt.Println(*p) // read i through the pointer p
*p = 21         // set i through the pointer p
```
与C不同，Go没有指针运算。
```go
package main

import "fmt"

func main() {
	i, j := 42, 2701

	p := &i         // point to i
	fmt.Println(*p) // read i through the pointer
	*p = 21         // set i through the pointer
	fmt.Println(i)  // see the new value of i

	p = &j         // point to j
	*p = *p / 37   // divide j through the pointer
	fmt.Println(j) // see the new value of j
}

```

## 结构体（Structs）
结构体是字段的集合。使用点访问结构字段。
```go
package main

import "fmt"

type Vertex struct {
	X int
	Y int
}

func main() {
	fmt.Println(Vertex{1, 2})
    
    v := Vertex{1, 2}
	v.X = 4
	fmt.Println(v.X)
}
```
### 结构体指针
可以通过结构指针访问结构字段。
```go
package main

import "fmt"

type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	p := &v
	p.X = 1e9
	fmt.Println(v)
}

```
当我们通过结构指针p访问结构的字段X，可以这样写（*p）.X。然而，这种表示法很麻烦，所以Go语言允许我们只写p.X，而不显式解引用。
```go
package main

import "fmt"

type Vertex struct {
	X, Y int
}

var (
	v1 = Vertex{1, 2}  // has type Vertex
	v2 = Vertex{X: 1}  // Y:0 is implicit
	v3 = Vertex{}      // X:0 and Y:0
	p  = &Vertex{1, 2} // has type *Vertex
)

func main() {
	fmt.Println(v1, p, v2, v3)
}

```
结构字面量通过设置其字段的值来表示新分配的结构值。您可以使用 Name: 语法设置部分字段的值。（命名字段的顺序无关紧要。）
特殊前缀&返回一个指向结构值的指针。

## Arrays
类型[n]T是n个类型为T的值的数组。 表达式 ： 
```go
var a [10]int
```
将变量a声明为一个由十个整数组成的数组。
数组的长度是其类型的一部分，因此无法调整数组的大小。这似乎有局限性，但不要担心；Go提供了一种使用数组的方便方法。
```go
package main

import "fmt"

func main() {
	var a [2]string
	a[0] = "Hello"
	a[1] = "World"
	fmt.Println(a[0], a[1])
	fmt.Println(a)

	primes := [6]int{2, 3, 5, 7, 11, 13}
	fmt.Println(primes)
}
```
## Slices
数组具有固定的大小。切片是动态数组。在实践中，切片比数组更常见。类型[]T是具有类型T的元素的slice 。切片是通过指定两个索引来形成的，一个是由冒号分隔的下界和上界：a[low : high]
这会截取一个包括第一个元素（low ）但不包括最后一个元素（high）的slice。
以下表达式创建一个切片，该切片包括的元素1到3：a[1:4]
```go
package main

import "fmt"

func main() {
	primes := [6]int{2, 3, 5, 7, 11, 13}

	var s []int = primes[1:4]
	fmt.Println(s)
}

```

### 切片类似于对数组的引用
切片不存储任何数据，它只描述底层数组的一部分。更改切片的元素会修改其底层数组的相应元素。共享相同底层阵列的其他切片将看到这些变化。
```go
package main

import "fmt"

func main() {
	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
	fmt.Println(names)

	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b)

	b[0] = "XXX"
	fmt.Println(a, b)
	fmt.Println(names)
}

```
### Slice字面量
切片就像一个没有长度的数组。
这是一个数组：
```go
[]bool{true, true, false}
```

这会创建与上面相同的数组，然后构建一个引用它的切片：
```go
[]bool{true, true, false}
```

```go
package main

import "fmt"

func main() {
	q := []int{2, 3, 5, 7, 11, 13}
	fmt.Println(q)

	r := []bool{true, false, true, true, false, true}
	fmt.Println(r)

	s := []struct {
		i int
		b bool
	}{
		{2, true},
		{3, false},
		{5, true},
		{7, true},
		{11, false},
		{13, true},
	}
	fmt.Println(s)
}

```
### Slice 默认值
使用切片时，可以省略上限或下限，而使用它们的默认值。对于下限，默认为零，对于上限，默认为切片长度。
对数组 var a [10]int，这些切片表达式是等效的：
```go
a[0:10]
a[:10]
a[0:]
a[:]
```
```go
package main

import "fmt"

func main() {
	s := []int{2, 3, 5, 7, 11, 13}

	s = s[1:4]
	fmt.Println(s)

	s = s[:2]
	fmt.Println(s)

	s = s[1:]
	fmt.Println(s)
}

```
### slice长度和容量
切片既有长度也有容量。切片的长度是它包含的元素个数。片的容量是底层数组中的元素数，从切片中的第一个元素开始计数。切片的长度和容量s可以使用表达式len（s）和cap（s）来获得。
```go
package main

import "fmt"

func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	printSlice(s)

	// Slice the slice to give it zero length.
	s = s[:0]
	printSlice(s)

	// Extend its length.
	s = s[:4]
	printSlice(s)

	// Drop its first two values.
	s = s[2:]
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

```
您可以通过重新切片来延长切片的长度，前提是它有足够的容量。尝试更改示例程序中的一个切片操作，以将其扩展到其容量之外，然后看看会发生什么。
### Slice零值
切片的零值为nil。值位nil的切片长度和容量为0，并且没有底层数组。
```go
package main

import "fmt"

func main() {
	var s []int
	fmt.Println(s, len(s), cap(s))
	if s == nil {
		fmt.Println("nil!")
	}
}

```
### 使用make创建切片
切片可以使用内置的make函数创建；这就是创建动态大小数组的方法。make函数分配一个空数组，并返回一个引用该数组的切片：
```go
a := make([]int, 5)  // len(a)=5
```
要指定容量，请传递make函数的第三个参数以：
```go
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```
```go
package main

import "fmt"

func main() {
	a := make([]int, 5)
	printSlice("a", a)

	b := make([]int, 0, 5)
	printSlice("b", b)

	c := b[:2]
	printSlice("c", c)

	d := c[2:5]
	printSlice("d", d)
}

func printSlice(s string, x []int) {
	fmt.Printf("%s len=%d cap=%d %v\n",
		s, len(x), cap(x), x)
}

```
### 二维slice
切片可以包含任何类型，包括其他切片。
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	// Create a tic-tac-toe board.
	board := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	}

	// The players take turns.
	board[0][0] = "X"
	board[2][2] = "O"
	board[1][2] = "X"
	board[1][0] = "O"
	board[0][2] = "X"

	for i := 0; i < len(board); i++ {
		fmt.Printf("%s\n", strings.Join(board[i], " "))
	}
}

```
### Append
将新元素附加到切片中是很常见的，因此Go提供了一个内置函数append 。
```go
func append(s []T, vs ...T) []T
//append的第一个参数s是T类型的切片，其余的是要附加到切片的T值。
```
append的结果值是一个切片，包含原始切片的所有元素加上提供的值。
如果s的底层数组太小，无法容纳所有给定的值，则会分配一个更大的数组。返回的切片将指向新分配的数组。
```go
package main

import "fmt"

func main() {
	var s []int
	printSlice(s)

	// append works on nil slices.
	s = append(s, 0)
	printSlice(s)

	// The slice grows as needed.
	s = append(s, 1)
	printSlice(s)

	// We can add more than one element at a time.
	s = append(s, 2, 3, 4)
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

```
### Range
for循环的range形式在切片或映射上进行迭代。在切片上进行遍历时，每次迭代都会返回两个值。第一个是索引，第二个是该索引处元素的副本。
```go
package main

import "fmt"

var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {
	for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
	}
}

```
您可以用 _ 来跳过索引或值。如果只需要索引，可以省略第二个变量。
```go
for i, _ := range pow
for _, value := range pow
for i := range pow
```
```go
package main

import "fmt"

func main() {
	pow := make([]int, 10)
	for i := range pow {
		pow[i] = 1 << uint(i) // == 2**i
	}
	for _, value := range pow {
		fmt.Printf("%d\n", value)
	}
}

```
### 练习：切片
实现 Pic函数.它应该返回一个长度为dy的切片，其中的每个元素都是dx 8位无符号整数的切片. 当你运行该程序时, 它将显示您的图片，将整数解释为灰度值.
图像的选择取决于您。有趣的函数包括（x+y）/2、x*y和x^y
您需要使用一个循环来分配[][]uint8中的每个[]uint8。
(使用uint8（intValue）在类型之间进行转换。)

_一下代码仅供参考_
```go
package main

import "golang.org/x/tour/pic"
func Pic(dx, dy int) [][]uint8 {
    total := make([]uint8, dx*dy)
    res := make([][]uint8, dx)
    for x := 0; x < dx; x++ {
        res[x], total = total[0:dy], total[dy:]
        for y := 0; y < dy; y++ {
            res[x][y] = uint8(x * y)
        }
    }
    return res
}

func main() {
    pic.Show(Pic)
}
```
![image.png](https://cdn.nlark.com/yuque/0/2023/jpeg/12487795/1683790391324-b538633e-0cdf-4fe6-9685-d3bead652def.jpeg#averageHue=%237d7dff&clientId=u2accdf68-48c2-4&from=paste&id=u1e9ab2cb&originHeight=256&originWidth=256&originalType=url&ratio=1&rotation=0&showTitle=false&size=36760&status=done&style=none&taskId=u8528cc59-8143-4c71-8db6-0cbce242075&title=)


## Maps
Map 是键值对的映射
map的零值为nil。nil map 没有key，也不能向其添加key。
make函数返回给定类型的map，该map已初始化并可以使用。
```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex

func main() {
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{
		40.68433, -74.39967,
	}
	fmt.Println(m["Bell Labs"])
}

```
map 赋值与结构赋值类似，但key是必需的。
```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}

func main() {
	fmt.Println(m)
}

```
如果顶级类型只是一个类型名称，则可以从赋值的元素中省略它。
```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}

func main() {
	fmt.Println(m)
}

```
### 修改map
```go
package main

import "fmt"

func main() {
	m := make(map[string]int)

	m["Answer"] = 42
	fmt.Println("The value:", m["Answer"])

	m["Answer"] = 48
	fmt.Println("The value:", m["Answer"])

	delete(m, "Answer")
	fmt.Println("The value:", m["Answer"])

	v, ok := m["Answer"]
	fmt.Println("The value:", v, "Present?", ok)
}

```
在map m中插入或更新元素：
```go
m[key] = elem
```
查找元素：
```go
elem = m[key]
```
删除元素：
```go
delete(m, key)
```
通过接收第二个参数测试是否存在对应的key：
```go
elem, ok = m[key]
```
若key在m中，则ok为true。否则ok为false。
若key不在map中，则elem是map元素类型的零值。
注意：如果尚未声明elem或ok，则可以使用简短的声明形式：
```go
elem, ok := m[key]
```
### 练习：map
实现WordCount。它应该返回字符串s中每个“单词”计数的map。wc.Test函数针对所提供的函数运行一个测试套件，并打印成功或失败。你可能会用到 strings.Fields。
_一下代码仅供参考_
```go
package main

import (
	"strings"

	"golang.org/x/tour/wc"
)

func WordCount(s string) map[string]int {
	res := make(map[string]int)
	words := strings.Fields(s)
	for _, word := range words {
		res[word]++
	}
	return res
}

func main() {
	wc.Test(WordCount)
}

```
## 函数类型
函数也是值。它们可以像其他值一样传递。函数值可以用作函数参数和返回值。
```go
package main

import (
	"fmt"
	"math"
)

func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}

```
### 函数闭包
Go函数可能是闭包。闭包是一个函数值，它引用来自其主体外部的变量。函数可以访问并分配给引用的变量；从这个意义上说，函数是“绑定”到变量的。例如，addr函数返回一个闭包。每个闭包都绑定到它自己的sum变量。
```go
package main

import "fmt"

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}

```

### 练习：斐波那契闭包
让我们玩一下function。
实现一个fibonacci函数，它返回一个函数（闭包），该函数返回连续的fibonacci数（0，1，1，2，3，5，…）。
_一下代码仅供参考_
```go
package main

import "fmt"

// fibonacci is a function that returns
// a function that returns an int.
func fibonacci() func() int {
	var pre,cur = 0,0
	return func()int{
		temp := cur
		cur = pre+cur
		pre = temp
		if cur == 0 {
			cur = 1
			return 0
		}
		return temp
	}
}
func fibonacci1() func() int {
	var pre, cur, idx = 0, 1, -1
	return func() int {
		idx++
		if idx <= 1 {
			return idx
		}
		temp := cur
		cur = pre + cur
		pre = temp
		return cur
	}
}
func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}

```
