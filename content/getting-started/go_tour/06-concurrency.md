+++
title = "6、并发"
description = "Go提供的并发功能是核心语言的一部分"
weight = 6
url = "/tour/06.html"
+++

Go提供的并发功能是核心语言的一部分。
## Goroutines
goroutine是一个由Go运行时管理的轻量级线程。
```go
go f(x, y, z)
```
开始运行新的goroutine f(x, y, z)
f、x、y和z的求值发生在当前goroutine中，f的执行发生在新goroutine。
Goroutines在相同的地址空间中运行，因此对共享内存的访问必须同步。sync包提供了有用的并发原语，尽管Go中并不需要它们，因为还有其他基元。
```go
package main

import (
	"fmt"
	"time"
)

func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")
	say("hello")
}

```
## Channels
Channel是一种类型化的管道，通过它可以使用通道操作符<-发送和接收值。
```go
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and
           // assign value to v.
```
数据沿箭头方向流动。
与map和slice一样， channel必须在使用前初始化：
```go
ch := make(chan int)
```
默认情况下，发送和接收块，直到对方准备好为止。这允许goroutine在没有显式锁或条件变量的情况下进行同步。
```go
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)
}

```
示例代码对一个切片中的数字求和，将工作分配到两个goroutine之间。一旦两个goroutine都完成了计算，它就会计算出最终结果。
## Buffered Channels
通道可以进行缓冲。提供缓冲区长度作为初始化缓冲通道的第二个参数：
```go
ch := make(chan int, 100)
```
仅当缓冲区已满时发送到缓冲通道阻塞。缓冲区为空时接收阻塞。
修改示例以使缓冲区溢出，然后看看会发生什么。
```go
package main

import "fmt"

func main() {
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}

```
## Range and Close
发送方可以关闭一个通道，以指示不再发送任何值。接收器可以通过为接收表达式分配第二个参数来测试通道是否已关闭：如下
```go
v, ok := <-ch
```
如果没有更多的值可接收并且通道已关闭，则ok为false。
还可以通过 i：=range c 的循环重复接收来自通道的值，直到通道关闭，循环退出。
```go
package main

import (
	"fmt"
)

func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}

```
注意：

- 只有发送方应该关闭频道，而不是接收方。在关闭的频道上发送会引起panic。
- channel不像文件；通常不需要关闭它们。只有当接收器必须被告知没有更多的值到来时，例如终止range循环时，才需要关闭。
## Select
select语句允许goroutine等待多个通信操作。
一个select阻塞，直到它的一个case可以运行，然后它执行那个case。如果多个case准备好了，它会随机选择一个。
```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}

```

 如果没有其他案例准备就绪，则选择中的默认案例将运行。使用默认情况尝试在不阻止的情况下发送或接收：
```go
select {
case i := <-c:
    // use i
default:
    // receiving from c would block
}
```
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	tick := time.Tick(100 * time.Millisecond)
	boom := time.After(500 * time.Millisecond)
	for {
		select {
		case <-tick:
			fmt.Println("tick.")
		case <-boom:
			fmt.Println("BOOM!")
			return
		default:
			fmt.Println("    .")
			time.Sleep(50 * time.Millisecond)
		}
	}
}

```
## Exercise: Equivalent Binary Trees
练习：等价二叉树 代码已同步仅供参考 [github](https://github.com/LittleMoreInteresting/beginner-example/blob/master/lesson006/exercise-equivalent-binary-trees.go)

## sync.Mutex
 我们已经看到了channel对于goroutines之间的交流是多么的好。
但如果我们不需要通讯呢？如果我们只想确保一次只有一个goroutine可以访问一个变量以避免冲突，该怎么办？
这个概念被称为互斥，提供互斥的数据结构的传统名称是_mutex_。
Go的标准库通过sync.Mutex及其两种方法提供互斥：

- Lock
- Unlock

我们可以定义一个要在互斥中执行的代码块，方法是用Inc方法中所示的Lock和Unlock调用包围它。
我们还可以使用defer来确保互斥对象将像Value方法中那样被解锁。
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	mu sync.Mutex
	v  map[string]int
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mu.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mu.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}

```
## Exercise: Web Crawler（网络爬虫）
代码已同步仅供参考 [Github](https://github.com/LittleMoreInteresting/beginner-example/blob/master/lesson006/exercise-web-crawler.go)
