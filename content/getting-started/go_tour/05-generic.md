+++
title = "5、泛型"
description = "Go支持使用类型参数进行泛型编程。本课展示了在代码中使用泛型的一些示例。"
weight = 5
url = "/tour/05.html"
+++

> Go支持使用类型参数进行泛型编程。本课展示了在代码中使用泛型的一些示例。

## 类型参数
Go函数可以使用类型参数让函数支持多个类型。函数的类型参数用中括号包裹，位于函数的参数之前。
```go
func Index[T comparable](s []T, x T) int
```
这个声明意味着s是任何类型T的一个切片，它满足comparable内置约束。x也是相同类型的值。
compatible是一个有用的约束，可以使用==和！=类型值上的运算符。在本例中，我们使用它将一个值与所有切片元素进行比较，直到找到匹配。此Index函数适用于任何支持比较的类型。
```go
package main

import "fmt"

// Index returns the index of x in s, or -1 if not found.
func Index[T comparable](s []T, x T) int {
	for i, v := range s {
		// v and x are type T, which has the comparable
		// constraint, so we can use == here.
		if v == x {
			return i
		}
	}
	return -1
}

func main() {
	// Index works on a slice of ints
	si := []int{10, 20, 15, -10}
	fmt.Println(Index(si, 15))

	// Index also works on a slice of strings
	ss := []string{"foo", "bar", "baz"}
	fmt.Println(Index(ss, "hello"))
}

```
## 泛型类型
除了泛型函数外，Go还支持泛型类型。可以使用类型参数对类型进行参数化，这对于实现泛型数据结构非常有用。
```go
package main

// List represents a singly-linked list that holds
// values of any type.
type List[T any] struct {
	next *List[T]
	val  T
}

func main() {
}

```
此示例演示了一个简单的类型声明，用于包含任何类型值的单链列表。

