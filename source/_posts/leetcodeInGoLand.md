---
title: 'GoLand 的完美 leetcode 刷题体验'
date: 2022-08-17 22:32:14
tags: ['linux', 'go']
---

大概从大二开始，我开始陆陆续续地刷一些 leetcode 题，最开始是用 java，之后很长一段时间在用 go。在刷题的过程中，我的刷题方式发生了很大的变化。

最开始是使用 leetcode 的在线编辑器刷题，题目极其简单还好，稍微复杂一些需要引库，在线编辑器的自动补全能把人逼疯...没多久，我就弃用了这种最原始的刷题方法。

> 一个小提示：参加技术招聘时的笔试有很大可能是在网页编辑器中作答的。
>
> 平常不使用在线编辑器刷题主要是为了方便，但也不要因此变成离开 IDE 后就不知所措、标准库方法名都忘记的工具型选手！

<!--more-->

抛开在线编辑器，能想到的方法当然只有在本地编程了。最开始是 vscode + golang 拓展，每次都要：

1. 新建代码文件，命名为题目名称；
2. 打开 leetcode 题目描述，把代码模板粘贴下来；
3. 实现算法，复制粘贴到 leetcode 在线编辑器中，提交测试；
4. 如未通过则重复 3 。

重复性劳动总是让人烦躁，于是我找到了[ vscode 中的 leetcode 拓展](https://github.com/LeetCode-OpenSource/vscode-leetcode)，该拓展可在 vscode 中展示题目列表，选中题目后自动在 vscode 中预览题目并生成文件描述和模板到指定文件夹。这在很大程度上节省了我的工作量。

然而，随着某次 go 或 vscode 的更新，vscode 内对 go 的自动补全检查扩大到了文件夹级别。（或许是随着 gomod 的引入？我已经不太清楚了）因为不同题目全部在文件夹的同一级，所有的在不同题目中定义的结构体、函数只要重名就会报重声明错误，而这在一个存储上百道题解的文件夹中带来的是毁天灭地的灾难。我的源码文件近乎一半被标红，每每看到那一片片的红色波浪线都给我带来生理上的不适。

> 现在想来应该多去查查 leetcode 拓展的文档的，或许有什么解法也说不定？

时间过得飞快，转眼间就毕业工作了。虽然工作主要使用的语言是 python，但我平时还是习惯写写 go。前段时间几位同事说要比赛力扣连续打卡一个月，我也顺便参与了。这次从 vscode 换成了 goland，我打算一步到位配置一个完美的力扣刷题环境。

以下是我的配置过程及思路：（~~唠叨半天总算说到重点了。~~）

### 安装 leetcode 刷题插件

与 vscode 类似，GoLand 中也有类似的刷题插件，名为 LeetCode Editor。

### 组织源码文件夹

安装拓展完成后，可在设置中调整 leetcode 题目描述、源码模板下载到的位置。假设你把位置填成了 `/home/amtoaer/Documents/code/`。

随便点击一个题目，将会发现实际的目录结构如下：

```sh
# 此时位于 /home/amtoaer/Documents/code/
./leetcode/
└── editor
    ├── cn
    │   └── doc
    │       └── content
    │           └── 1302.层数最深叶子节点的和
    └── en
```

也就是说，如果使用中文刷题，并把拓展配置中的`TempFilePath`设置为`$DIR`的话，实际的源码所在目录为`$DIR/leetcode/editor/cn/`，并且该目录下还会有一个多余的`doc`文件夹用于存储题目描述。

如果有保存源码文件的需求，请到该目录下寻找。

如果你像我一样喜欢把题目上传到 git 仓库，在`/leetcode/editor/cn/`中新建 git 仓库并在该文件夹下新建`.gitignore`，于其中写入一行：

```sh
doc
```

将题目描述文档所在的文件夹忽略即可。

### 解决同文件夹下的命名冲突错误

通过查阅相关资料，我得知在 go 源码文件中做以下任意两个操作，可使 go build 忽略该文件信息：

1. 在源码文件名前加入下划线`_`；
2. 在源码文件内写入 `//go:build ignore`。

然而，这两种方式都有致命缺陷，即在 GoLand 的编辑器头部有无法关闭的醒目标识，这对于完美主义的我们是不能接受的。

继续查找资料，发现 leetcode editor 拓展是支持自定义代码文件模板和路径的。且[ #304 ](https://github.com/shuzijun/leetcode-editor/issues/304)提到可通过在自定义文件名中加入路径分割符来新建文件夹。

既然平铺的源码文件会导致重声明错误，那我们可以给每个源码文件都创建一个新的文件夹。

首先勾选`Custom Template`选项，接着将`Code FileName`修改为：

```sh
$!{question.frontendQuestionId}.${question.title}/solution_test
```

这将使文件结构变为：

```sh
├── 102.二叉树的层序遍历
│   └── solution_test.go
├── 103.二叉树的锯齿形层序遍历
│   └── solution_test.go
├── 104.balabala
│   └── solution_test.go
```

这时我们已经做到了单个代码文件的命名隔离。

### 易于调试

得益于拓展的自定义代码模板功能与区域代码提交机制，我们可以非常简单地调试我们的代码。

> 区域代码提交：指拓展维护两条固定注释，分别作为提交位置的起点和终点，提交时只有这两个注释之间的代码会被提交。
>
> 我们可以把调试部分（如单测或者 main 函数）放在提交区域外，这样不会对提交的代码有任何影响。

一个简易的模板如下图，需将其填写在 leetcode plugin 配置中的`Code Template`项：

```go
package leetcode

import(
    "testing"
)

${question.content}

${question.code}

func Test$!velocityTool.camelCaseName(${question.titleSlug})(t *testing.T){
    
}
```

这样只需要把样例输入输出写在测试函数内，点击运行即可。

一个依此模板生成的文件如下：

```go
package leetcode

import(
    "testing"
)

//给你一个正整数数组 nums，请你帮忙从该数组中找出能满足下面要求的 最长 前缀，并返回该前缀的长度： 
//
// 
// 从前缀中 恰好删除一个 元素后，剩下每个数字的出现次数都相同。 
// 
//
// 如果删除这个元素后没有剩余元素存在，仍可认为每个数字都具有相同的出现次数（也就是 0 次）。 
//
// 
//
// 示例 1： 
//
// 
//输入：nums = [2,2,1,1,5,3,3,5]
//输出：7
//解释：对于长度为 7 的子数组 [2,2,1,1,5,3,3]，如果我们从中删去 nums[4] = 5，就可以得到 [2,2,1,1,3,3]，里面每个数
//字都出现了两次。
// 
//
// 示例 2： 
//
// 
//输入：nums = [1,1,1,2,2,2,3,3,3,4,4,4,5]
//输出：13
// 
//
// 
//
// 提示： 
//
// 
// 2 <= nums.length <= 10⁵ 
// 1 <= nums[i] <= 10⁵ 
// 
//
// Related Topics 数组 哈希表 👍 64 👎 0


//leetcode submit region begin(Prohibit modification and deletion)
func maxEqualFreq(nums []int) int {

}
//leetcode submit region end(Prohibit modification and deletion)


func TestMaximumEqualFrequency(t *testing.T){
    
}
```

在文件中实现算法、并在本地调试的例子如图所示：

![](https://cdn.u1.huluxia.com/g4/M01/7F/5A/rBAAdmL9Ho-ATTS6AAmiD_GXDZ0282.png)

至此该方案已经比较完美了。

### 旧代码仓库的迁移

原有的代码虽然不会继续调试了，但仍然有保留价值。我选择把它们整理成相同的目录结构。

1. 按上述流程设置 leetcode editor 插件。

2. 克隆原有的平铺型代码仓库，将所有的代码以及`.git`文件夹整个移动到`$DIR/leetcode/editor/cn/`。

3. 写一个简单的`python`脚本用于迁移源码：

   ```python
   import os
   
   files = os.listdir('.')
   
   for file in files:
       items = file.rsplit('.', 1)
       if len(items) != 2:
           continue
       name, extension = items
       if extension != 'go':
           continue
       os.mkdir(name)
       os.rename(file, f'{name}/solution_test.go')
   ```

4. `git add` & `git commit` & `git push`
