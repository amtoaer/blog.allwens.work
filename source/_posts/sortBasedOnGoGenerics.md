---
title: 基于 Go 泛型的排序包
date: 2022-02-03 13:11:43
tags: ['go'] 
---

接[上篇](https://blog.allwens.work/goGenerics/)，最近发现了一个深入讲解泛型的仓库：[go-generics-the-hard-way](https://github.com/akutz/go-generics-the-hard-way)。简单阅读过后，我总算实现了多类型 Slice 的排序。（我菜死了！）

<!--more-->

## 理论基础

**一、可通过定义类型集合来进行泛型约束。**

```go
// Numeric expresses a type constraint satisfied by any numeric type.
type Numeric interface {
	uint | uint8 | uint16 | uint32 | uint64 |
		int | int8 | int16 | int32 | int64 |
		float32 | float64 |
		complex64 | complex128
}

// Sum returns the sum of the provided arguments.
func Sum[T Numeric](args ...T) T {
	var sum T
	for i := 0; i < len(args); i++ {
		sum += args[i]
	}
	return sum
}
```

**二、波浪线前缀在约束中表示支持同基础类型的其它类型。**在定义上述内容的前提下，编写以下代码：

```go
// id is a new type definition for an int64
type id int64

func main() {
	fmt.Println(Sum([]id{1, 2, 3}...))
}
```

编译将会报错：

> id does not implement Numeric (possibly missing ~ for int64 in constraint Numeric)

需在类型集合的`int64`部分加入`~`前缀。

**三、当某类型中包含泛型时，泛型符号必须被包含在函数接收者中。**如有以下泛型类型：

```go
// Ledger is an identifiable, financial record.
type Ledger[T ~string, K Numeric] struct {

	// ID identifies the ledger.
	ID T

	// Amounts is a list of monies associated with this ledger.
	Amounts []K

	// SumFn is a function that can be used to sum the amounts
	// in this ledger.
	SumFn SumFn[K]
}
```

为其定义方法时，需在函数接收者部分显式包含泛型符号：

```go
// PrintIDAndSum emits the ID of the ledger and a sum of its amounts on a
// single line to stdout.
func (l Ledger[T, K]) PrintIDAndSum() {
	fmt.Printf("%s has a sum of %v\n", l.ID, l.SumFn(l.Amounts...))
}
```

## 排序实现

了解上述知识后，批量实现`sort.Interface`就很简单了，全部代码如下：

```go
package sort

type comparable interface {
	~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
		~int | ~int8 | ~int16 | ~int32 | ~int64 |
		~float32 | ~float64 |
		~string
}

// Sortable a generics slice which implements sort.Interface
type Sortable[T comparable] []T

func (s Sortable[T]) Len() int {
	return len(s)
}

func (s Sortable[T]) Swap(i, j int) {
	s[i], s[j] = s[j], s[i]
}
func (s Sortable[T]) Less(i, j int) bool {
	return s[i] < s[j]
}
```

对各类型切片排序时，简单使用`Sortable`封装即可：

```go
package main

import (
	"fmt"
	"sort"

	gsort "github.com/amtoaer/generic-sort"
)

func main() {
	intSlice := []int{1, 3, 2, 4}
	stringSlice := []string{"h", "e", "l", "l", "o"}
	byteSlice := []byte{'h', 'e', 'l', 'l', 'o'}
	sort.Sort(gsort.Sortable[int](intSlice))
	sort.Sort(gsort.Sortable[string](stringSlice))
	sort.Sort(gsort.Sortable[byte](byteSlice))
	fmt.Println(intSlice)    // [1 2 3 4]
	fmt.Println(stringSlice) // [e h l l o]
	fmt.Println(byteSlice)   // [101 104 108 108 111]
}
```

