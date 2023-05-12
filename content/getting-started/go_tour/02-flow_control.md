+++
title = "基础篇（二）：Go语言流程控制"
description = "基础篇（二）：Go语言流程控制"
weight = 2
url = "/tour/02.html"
+++
## For
Go只有一个循环结构，即for循环。
```go
package main

import "fmt"

func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}
```
基本的for循环有三个用分号分隔的部分：

- 初始化语句：在第一次迭代之前执行
- 条件表达式：在每次迭代之前求值
- 后置语句：在每次循环结束时执行

初始化语句通常是一个简短的变量声明，在那里声明的变量只在for语句的范围内可见。
一旦条件表达式的计算结果为false，循环将停止迭代。
注意：与C、Java或JavaScript等其他语言不同，for语句的三个组件周围没有括号，并且必须要有大括号{}。
初始化语句和后置语句是可选的。如下：
```go
package main

import "fmt"

func main() {
	sum := 1
	for ; sum < 1000; {
		sum += sum
	}
	fmt.Println(sum)
}
```
更进一步，还可以去掉分号：类似C语言中的的while那样，在Go中是用for实现的。如下：
```go
package main

import "fmt"

func main() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}

```
如果省略循环条件，它将永远循环，因此一个死循环就可以这样写：
```go
package main

func main() {
	for {
        // Do something forever
	}
}

```
## If
Go的if语句类似于它的for循环；表达式不需要用括号（）括起来，但必须要有大括号｛｝。
```go
package main

import (
	"fmt"
	"math"
)

func sqrt(x float64) string {
	if x < 0 {
		return sqrt(-x) + "i"
	}
	return fmt.Sprint(math.Sqrt(x))
}

func main() {
	fmt.Println(sqrt(2), sqrt(-4))
}

```

与for一样，if语句可以在条件之前以执行一个短语句。语句声明的变量只能在if结束之前使用。如下：
```go
package main

import (
	"fmt"
	"math"
)

func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
	return lim
}

func main() {
	fmt.Println(
		pow(3, 2, 10),
		pow(3, 3, 20),
	)
}

```
（请尝试在最后一个return(line 12)语句中使用v。）
### If and else
在if短语句中声明的变量在任何else块中都可用。
```go
package main

import (
	"fmt"
	"math"
)

func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%g >= %g\n", v, lim)
	}
	// can't use v here, though
	return lim
}

func main() {
	fmt.Println(
		pow(3, 2, 10),
		pow(3, 3, 20),
	)
}

```
（在main中对fmt.Println的调用之前，会先对两个pow调用求值。）
### 练习：循环和函数
为了练习函数和方法，让我们实现一个平方根函数：
给定一个数字x，找到z²最接近x的数字z。
计算机通常使用循环计算x的平方根。从一些可能的z开始，我们可以根据z²与x的接近程度来调整z，从而产生更好的结果：
```latex
z -= (z*z - x) / (2*z)
```
重复这种调整会使结果变得越来越好，直到我们得出尽可能接近实际平方根的答案
在函数Sqrt中实现这一功能。z的一个不错的起始值是1，无论输入是什么。首先，重复计算10次，并在计算过程中打印每个z。看看你离x（1，2，3，…）的各种值的答案有多近，以及猜测改进的速度有多快。
提示：要声明和初始化浮点值，请定义为浮点类型或使用类型转换转换：
```go
z := 1.0
z := float64(1)
```
接下来，一旦值停止更改（或仅更改很小的量），就将循环条件更改为停止。看看这是多于还是少于10次迭代。尝试对z进行其他初始猜测，如x或x/2。您的函数的结果与标准库中的math.Sqrt有多接近？
（注意：如果你对算法的细节感兴趣，上面的z²−x是z²离它需要的位置（x）有多远，除以2z是z²的导数，通过z²的变化速度来衡量我们调整z的程度。这种通用方法被称为牛顿方法。它适用于许多函数，但特别适用于平方根。）
_以下代码仅供参考_
```go

import (
	"fmt"
	"math"
)

func Sqrt(x float64) float64 {
	var z = 1.0
	var old float64
	for math.Abs(old-z) >= 0.000001 {
		old = z
		z -= (z*z - x) / (2 * z)
	}
	return z
}

func main() {
	fmt.Println(Sqrt(2), math.Sqrt(2))
	fmt.Println(Sqrt(10), math.Sqrt(10))
}

```
## Switch
switch语句是编写if-else的一种更简短的方法。它执行第一个满足条件的分支。
```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
}
```
Go的switch结构与C、C++、Java、JavaScript和PHP中的开关类似，只是Go只运行第一个符合条件的分支，而不是随后的所有案例。实际上，在Go中在每个分支最后都加了break语句。另一个重要的区别是Go的switch的分支条件不一定是常数，所涉及的值也不需要是整数。
### 执行顺序
switch 分支是从上到下进行判断的，当某一个分支成功时停止向下执行。例如：
```go
switch i {
case 0:
case f():
}
// 如果i = 0 f函数是不执行的
```
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("When's Saturday?")
	today := time.Now().Weekday()
	switch time.Saturday {
	case today + 0:
		fmt.Println("Today.")
	case today + 1:
		fmt.Println("Tomorrow.")
	case today + 2:
		fmt.Println("In two days.")
	default:
		fmt.Println("Too far away.")
	}
}
```
## 无条件Switch
无条件Switch等同于 Switch true。这种构造可以用来编写长if-then-else链。
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
}
```

## Defer
defer语句将一个函数的执行延迟到锁住函数返回时。延迟调用的参数会立即求值，但在周围的函数返回之前不会执行函数调用。
```go
package main

import "fmt"

func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}
```
defer的函数调用被推送到堆栈中。当函数返回时，其defer调用将按后进先出的顺序执行。
```go
package main

import "fmt"

func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}

```
