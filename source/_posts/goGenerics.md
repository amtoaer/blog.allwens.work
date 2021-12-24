---
title: Go 泛型初探
date: 2021-12-24 18:38:04
tags: ['go']
---

Go 团队于 12 月 14 日发布了 Go 1.18 Beta 1，正式引入了对泛型的支持。详见 [The Go Blog](https://go.dev/blog/go1.18beta1)。笔者近日也下载尝了个鲜。

## 下载安装

按照官方文档下载安装 beta 版本：

```bash
❯ go install golang.org/dl/go1.18beta1@latest

# 注：以下可执行文件位于$GOPATH/bin，如未配置环境变量可手动指定路径
❯ go1.18beta1 download 

❯ go1.18beta1 version
go version go1.18beta1 linux/amd64
```

安装完成！

<!-- more -->

## 简单体验

参考[官方的泛型文档](https://go.dev/doc/tutorial/generics)，我们可以轻松写出一个累加任意可加 slice 的方法：

![image-20211224190117034](https://rmt.ladydaily.com/fetch/allwens-work/storage/20211224190211.png)

由于 Go 的工具链还未更新适配新语法，vscode 会提示语法错误，不过实际运行的输出结果符合预期。

## 深入理解

想必很多人都想进一步了解 Go 新泛型的语法和用法，可遗憾的是官方似乎只给出了上节提到的简单文档，没有更详尽的解释了。目前我只找到了一些[第三方的泛型示例](https://github.com/mattn/go-generics-example)，感兴趣的可以阅读代码加深理解。

## 个人感想

或许是因为文档的欠缺，单单看示例让我很难想象到泛型的实际用处。本来拿到泛型后最想尝试的是实现多类型 slice 的排序，即批量实现`sort.Interface`：

```go
type Interface interface {
	// Len is the number of elements in the collection.
	Len() int

	// Less reports whether the element with index i
	// must sort before the element with index j.
	//
	// If both Less(i, j) and Less(j, i) are false,
	// then the elements at index i and j are considered equal.
	// Sort may place equal elements in any order in the final result,
	// while Stable preserves the original input order of equal elements.
	//
	// Less must describe a transitive ordering:
	//  - if both Less(i, j) and Less(j, k) are true, then Less(i, k) must be true as well.
	//  - if both Less(i, j) and Less(j, k) are false, then Less(i, k) must be false as well.
	//
	// Note that floating-point comparison (the < operator on float32 or float64 values)
	// is not a transitive ordering when not-a-number (NaN) values are involved.
	// See Float64Slice.Less for a correct implementation for floating-point values.
	Less(i, j int) bool

	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```

写了老半天也没能成功，就放弃了...如果有读者了解的话，还望在评论区不吝赐教。

希望能尽快看到 Go 官方的详细泛型文档吧！
