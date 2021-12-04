---
title: Go中sync.cond的应用场景（含举例及对比）
date: 2021-12-04 12:22:19
tags: ['go']
---



最近在学习[Go语言高性能编程](https://geektutu.com/post/high-performance-go.html)时看到了`sync.Cond`条件变量这个概念，一时有些难以理解。查阅资料后对其有了些认知，特此记录。

## 概念

`sync`包提供了条件变量类型`sync.Cond`，它可以和互斥锁或读写锁组合使用，用来协调想要访问共享变量的协程。其主要作用机制是在对应的共享资源状态发生变化时，通知其它因此而阻塞的协程。

<!--more-->

## 定义

`sync.Cond`是一个结构体，其定义如下：

```go
// Each Cond has an associated Locker L (often a *Mutex or *RWMutex),
// which must be held when changing the condition and
// when calling the Wait method.
//
// A Cond must not be copied after first use.
type Cond struct {
        noCopy noCopy

        // L is held while observing or changing the condition
        L Locker

        notify  notifyList
        checker copyChecker
}
```

该类型可通过`sync`包中的`func NewCond(l Locker) *Cond`方法创建。

它有如下三个方法：

+ `func (c *Cond) Broadcast()`

  唤醒所有等待 c 的协程。

+ `func (c *Cond) Signal()`

  唤醒单个等待 c 的协程（如果有的话）。

+ `func (c *Cond) Wait()`

  自动解锁 `c.L`并暂停该协程执行，直到被唤醒。该函数在返回前会重新对`c.L`加锁。

## 应用场景

由以上描述可知，该结构的主要用法应该是多个协程调用`Wait`等待某个协程运行，该协程任务完毕后调用`Broadcast`或`Signal`将等待的协程唤醒。即：**多用在多个协程等待，一个协程通知的场景**。

举个生活中的小例子：

多位同学到达食堂，此时食堂还未做完饭，同学们需要等待食堂出餐后才能恰饭。

用`Go`条件变量对以上场景进行描述，可写出如下代码：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var (
	ok      = false                           // 是否做完饭
	food    = ""                              // 饭的名字
	rwMutex = &sync.RWMutex{}                 // 读写锁，用于锁定饭名的修改
	cond    = sync.NewCond(rwMutex.RLocker()) // 条件变量使用读写锁中的读锁
)

func makeFood() {
	// 做饭使用写锁（当然因为只有一个做饭协程，该锁并无实际意义）
	rwMutex.Lock()
	defer rwMutex.Unlock()
	fmt.Println("食堂开始做饭！")
	time.Sleep(3 * time.Second)
	ok = true
	food = "鱼香肉丝"
	fmt.Println("食堂做完饭了！")
	cond.Broadcast()
}

func waitToEat() {
	cond.L.Lock()
	defer cond.L.Unlock()
	for !ok {
		cond.Wait()
	}
	fmt.Println("总算吃到饭了，这顿吃的是", food)
}

func main() {
	for i := 0; i < 3; i++ {
		go waitToEat()
	}
	go makeFood()
	time.Sleep(4 * time.Second)
}
```

运行该程序，结果如下：

```
食堂开始做饭！
食堂做完饭了！
总算吃到饭了，这顿吃的是 鱼香肉丝
总算吃到饭了，这顿吃的是 鱼香肉丝
总算吃到饭了，这顿吃的是 鱼香肉丝
```



## 与其它概念的区别

### channel

众所周知，`channel`同样是 Go 中用于协程同步的概念，但与`sync.Cond`不同，`channel`多用于一对一通信，如果继承上面的例子，这次的场景应该是：

一位同学到达食堂，此时食堂还未做完饭，该同学需要等待食堂出餐后才能恰饭。

用`channel`描述为：

```go
package main

import (
	"fmt"
	"time"
)

var (
	food = ""
	ch   = make(chan struct{})
)

func makeFood() {
	fmt.Println("食堂开始做饭！")
	time.Sleep(3 * time.Second)
	food = "鱼香肉丝"
	fmt.Println("食堂做完饭了！")
	ch <- struct{}{}
}

func waitToEat() {
	<-ch
	fmt.Println("总算吃到饭了，这顿吃的是", food)
}

func main() {
	go waitToEat()
	go makeFood()
	time.Sleep(4 * time.Second)
}
```

运行结果为：

```
食堂开始做饭！
食堂做完饭了！
总算吃到饭了，这顿吃的是 鱼香肉丝
```

### sync.WaitGroup

`sync.WaitGroup`同样用于协程同步，但应用场景与`sync.Cond`刚好相反，后者多用于**多协程等待，单协程通知**，而前者多用于**单协程等待多协程执行完毕**。

仍然使用上述例子：

一位同学到达食堂，此时食堂多个窗口均未做完饭，该同学想要等待每个窗口都做完饭后各点一份吃。（多少有点离谱了XD）

代码描述如下：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var (
	wg sync.WaitGroup
)

func makeFood() {
	fmt.Println("窗口开始做饭！")
	time.Sleep(3 * time.Second)
	fmt.Println("窗口做完饭了！")
	wg.Done()
}

func waitToEat() {
	wg.Wait()
	fmt.Println("oho，每个窗口来一份！")
}

func main() {
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go makeFood()
	}
	go waitToEat()
	time.Sleep(4 * time.Second)
}
```

运行结果：

```
窗口开始做饭！
窗口开始做饭！
窗口开始做饭！
窗口做完饭了！
窗口做完饭了！
窗口做完饭了！
oho，每个窗口来一份！
```

## 结语

以上为个人的浅薄理解，如有错误欢迎在评论区指出，感谢！
