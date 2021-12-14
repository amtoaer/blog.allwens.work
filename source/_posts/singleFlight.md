---
title: singleflight的作用、实现及思考
date: 2021-12-14 17:31:29
tags: ['go']
---

最近学习实现了 [GeeCache](https://geektutu.com/post/geecache.html) 中的`singleflight`，写篇文章谈谈自己的理解。

## 是什么？

首先介绍一下缓存击穿的概念：

> 一个存在的 key，在缓存过期的一刻，同时有大量的请求，这些请求都会击穿到 DB ，造成瞬时 DB 请求量大、压力骤增。

其实很好理解，将缓存简单理解成`map[string]interface{}`，`get(key)`主要分为三步：

1. 检查 key 是否存在于 map 中，如存在则直接返回
2. key 不存在，调用`fn(key)`从数据库中获取数据
3. 调用完成，数据库返回结果，将返回的结果缓存到 map 中并返回

<!--more-->

当出现瞬时大量请求且 key 不存在于 map 中时，第一个请求会走到步骤二调用`fn(key)`访问数据库，在第一个请求的`fn(key)`还未返回时，后续请求到达。函数调用完成后才能缓存结果，但此时函数还未返回，所以后续请求同样会看到 key 不存在于缓存中，继续调用`fn(key)`访问数据库，最终导致大量请求直接落到数据库，就像缓存被击穿一样。

如何解决这个问题？一个很直接的想法是让后续请求“察觉”到此时`fn`正在调用，让后续请求不要重复调用，等待此时存在的`fn`返回结果即可。这就是`singleflight`做到的事情。

## 如何做？

我们首先参考 groupcache 中的实现：

```go
// Package singleflight provides a duplicate function call suppression
// mechanism.
package singleflight

import "sync"

// call is an in-flight or completed Do call
type call struct {
	wg  sync.WaitGroup
	val interface{}
	err error
}

// Group represents a class of work and forms a namespace in which
// units of work can be executed with duplicate suppression.
type Group struct {
	mu sync.Mutex       // protects m
	m  map[string]*call // lazily initialized
}

// Do executes and returns the results of the given function, making
// sure that only one execution is in-flight for a given key at a
// time. If a duplicate comes in, the duplicate caller waits for the
// original to complete and receives the same results.
func (g *Group) Do(key string, fn func() (interface{}, error)) (interface{}, error) {
	g.mu.Lock()
	if g.m == nil {
		g.m = make(map[string]*call)
	}
	if c, ok := g.m[key]; ok {
		g.mu.Unlock()
		c.wg.Wait()
		return c.val, c.err
	}
	c := new(call)
	c.wg.Add(1)
	g.m[key] = c
	g.mu.Unlock()

	c.val, c.err = fn()
	c.wg.Done()

	g.mu.Lock()
	delete(g.m, key)
	g.mu.Unlock()

	return c.val, c.err
}
```

实现非常简单，将一次函数调用抽象为`call`结构体，其中保存了函数调用的返回结果`val`和`err`，以及一个用于实现“单例”的`sync.WaitGroup`。

`Group`是实现非重复调用的核心，内建了 key 到函数调用的映射，以及保护映射的互斥锁。

在调用`Do`方法时：

1. 懒加载映射
2. 查看 key 对应的函数调用是否存在，如果已经存在则直接等待函数返回结果
3. 不存在则初始化一个新的函数调用，将其保存到映射中后调用函数，函数调用完成后删除映射

在这段代码中，`sync.WaitGroup`使用的尤其巧妙。我在[上篇文章](https://blog.allwens.work/sync-cond/)有提过：

> `sync.WaitGroup` 同样用于协程同步，但应用场景与 `sync.Cond` 刚好相反，后者多用于**多协程等待，单协程通知**，而前者多用于**单协程等待多协程执行完毕**。

而在此处，作者通过灵活使用`sync.WaitGroup`，达到了类似于`sync.Cond`的效果，堪称优雅。

## 有什么问题？

上述代码在`fn`正常返回的情况下不会有任何问题，但我们不得不考虑异常情况，如果`fn`执行遇到问题呢？

考虑一种场景，`fn`由于若干原因迟迟未返回，那么会有大量请求阻塞在`c.wg.Wait()`位置，这可能会导致：

+ 协程数量暴增
+ 内存使用暴涨
+ ……

如何解决？我们可以参考[官方的实现](https://pkg.go.dev/golang.org/x/sync/singleflight)。可以看到官方的拓展版本里，为`Group`拓展了两个公开方法：

+ `func (g *Group) DoChan(key string, fn func() (interface{}, error)) <-chan Result`

  > DoChan is like Do but returns a channel that will receive the results when they are ready.
  >
  > The returned channel will not be closed.

  DoChan 类似 Do，但会返回一个当结果就绪时收到结果的channel。

  返回的channel不会被关闭。

+ `func (g *Group) Forget(key string)`

  > Forget tells the singleflight to forget about a key. Future calls to Do for this key will call the function rather than waiting for an earlier call to complete.

  Forget告诉 singleflight 遗忘一个 key。将来对该 key Do 的调用会调用这个函数，而不是等待先前的调用完成。

前者 DoChan 可以很好地解决上述问题：因为返回的结果是 channel 而不是值，用户可以对其做超时控制，防止请求长时间阻塞：

```go
ch := g.DoChan(key, func() (interface{}, error) {
    ...
    return result, err
})

timeout := time.After(500 * time.Millisecond)

select {
case <-timeout:
        // 超时
    return
case <-ch:
    // 返回结果
}
```

而后者的主要应用场景，我在[sync.singleflight 到底怎么用才对？](https://www.cyningsun.com/01-11-2021/golang-concurrency-singleflight.html)找到了答案：

> 在一些对可用性要求极高的场景下，往往需要一定的请求饱和度来保证业务的最终成功率。一次请求还是多次请求，对于下游服务而言并没有太大区别，此时使用 `singleflight` 只是为了降低请求的数量级，那么使用 Forget () 提高下游请求的并发:
>
> ```go
> v, _, shared := g.Do(key, func() (interface{}, error) {
>     go func() {
>         time.Sleep(10 * time.Millisecond)
>         fmt.Printf("Deleting key: %v\n", key)
>         g.Forget(key)
>     }()
>     ret, err := find(context.Background(), key)
>     return ret, err
> })
> ```
>
> 当有一个并发请求超过 10ms，那么将会有第二个请求发起，此时只有 10ms 内的请求最多发起一次请求，即最大并发：100 QPS。单次请求失败的影响大大降低。

## 参考资料

以下顺序不分先后：

+ [动手写分布式缓存 - GeeCache 第六天 防止缓存击穿](https://geektutu.com/post/geecache-day6.html)
+ [sync.singleflight 到底怎么用才对？](https://www.cyningsun.com/01-11-2021/golang-concurrency-singleflight.html)
+ [singleflight.go - groupcache](https://github.com/golang/groupcache/blob/master/singleflight/singleflight.go)
+ [singleflight.go - x/sync](https://cs.opensource.google/go/x/sync/+/036812b2:singleflight/singleflight.go)
