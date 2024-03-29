---
title: 腾讯2021后台开发岗暑期实习面试记录
date: 2021-03-21 21:36:13
tags:
---

## 面试流程

腾讯的面试分为三部分：初试、复试和HR面试，前两轮是技术面，最后一轮主要是调研学生的个人理念、学习状况、家庭情况等内容。

三次面试通过后，几天内会收到腾讯HR助手发来的云证链接，主要内容是上传身份证和人脸信息，进行信息核实。接着静心等待oc及意向书即可。

## 个人情况

博主是东北大学计算机科学与技术专业的大三学生，了解Go、C++、Java、JavaScript等语言（主要使用Golang），绩点中下，简历上的项目写了Java课程设计的课设“Java画图板”、Web编程技术的课设“Java论坛系统”以及大创的项目“基于区块链的电子病历系统”，投递岗位为后台开发，无明确部门意向，base北京且接受调剂。

<!--more-->

## 时间线

 | 时间 | 事件                                                 |
 | ---- | ---------------------------------------------------- |
 | 3.17 | 投递简历                                             |
 | 3.18 | 约面试                                               |
 | 3.22 | 腾讯深圳一面                                         |
 | 3.30 | 腾讯深圳二面                                         |
 | 4.1  | 腾讯深圳二面挂掉                                     |
 | 4.7  | 腾讯北京捞人，约面试                                 |
 | 4.8  | 腾讯北京一面                                         |
 | 4.12 | 腾讯北京二面                                         |
 | 4.14 | 腾讯北京HR面                                         |
 | 4.15 | 收到oc（因为赶上了提前批最后一天，所以出结果比较快） |
 

-------

接下来是详细的面试流程。

# 第一次初试（1小时10分钟）

## 自我介绍

## 编程题目

开场是一道编程题，找到二叉树中两个结点的最近公共父节点。

这道题我曾经做过，但印象不够深刻，卡了比较久，且过程中小错误出现较多，痛苦面具。

> 面试官：
>   1. 需要学习一下Golang的命名规范。（然而我函数名写成test是因为不知道最近公共父节点的英文咋拼XD）
>   2. 省略掉函数中return后多余的else if和else（这里的确是自己考虑不周）

## 非编程题目

接着全部都是非编程题目了，以下分类记录。

### 策略题

一共64匹马，有8个赛道，最少通过多少次比赛可以得到最快的4匹马？（不允许计时，马的实力固定）

> 个人感觉这道题卡了我至少二十分钟，但最后也没想出来。
> 开始考虑了二进制，因为之前见到过一道形似的[老鼠试毒药问题](https://blog.csdn.net/z_qifa/article/details/74440959)。但面试官告诉我这题和二进制无关，于是我思路陷入了混沌...
> 面试官见这题我一时半会解决不了，就将其暂且搁置进行后续流程了。后续流程结束后，他又返回问了问我这道题的思路和答案，但最终我也没能解决。（面试官一直在引导我往正确的方向想，可惜我太不争气了呜呜呜）
> 网上题解：[https://blog.csdn.net/u013829973/article/details/80787928](https://blog.csdn.net/u013829973/article/details/80787928)

### 语言相关题

1. Golang的切片实现方式？既然切片底层是定长数组，那在切片中插入元素具体是如何处理的？切片作为函数参数传参时的考虑？

> 个人回答：Golang的切片是一个由指向底层数组的指针、容量、当前长度构成的结构体。
> 切片插入元素时，如果底层数组的容量足够，则直接length+1，切片中加入新增元素。如果底层数组容量不够，重新分配容量为原有容量二倍的底层数组，进行拷贝后令切片指向底层数组的指针指向新数组。
> 切片作为函数传参时，如果仅仅是修改切片内元素，因为在结构体值拷贝时可以拿到指向底层数组的指针，所以可以使用值传递。
> 当需要在切片中插入元素（即append）时，因为有可能会扩容进而导致底层数组指针的重新指向，所以需要传递原有切片的指针。

2. Golang的map是什么？发生哈希冲突的处理方法？

> 个人回答：Golang的map是哈希表。
> 发生哈希冲突时，Golang会将需要存入的value写入到key所对应的链表中（即使用通用的解决哈希冲突的[拉链法](https://blog.csdn.net/qq_32595453/article/details/80660676))。

3. Golang的结构体对interface是隐式实现的，如果将结构体赋值给它实现了的interface，通过该interface调用结构体内的方法，内部是怎样实现的？

> 个人对这一个问题了解不多，开始只是说结构体实现接口需要实现接口内的所有方法，面试官又强调了一次“内部实现”，即调用接口方法时，Golang内部应该有一个机制找到实际被赋值给接口的结构体内的对应方法。
> 我凭感觉回答了反射（reflect），他继续往深处问，我没办法，只能回答不太了解了。

### 个人问题

看到你简历里写了一些web开发经历，既然有web开发相关经验，为什么要投后台开发呢？（他介绍说腾讯内部也有大前端开发、全栈之类的岗位）

> 个人回答：我感觉自己的前端水平还很有限，只是能写而不是会写；因为自己其实大部分前端写的是Vue，对基础的Javascript/Typescript等了解反而不够深入，所以感觉自己没有足够能力胜任那样的职位。
> 现在回想来，说不定面试官觉得我同样没有能力胜任后台开发？（悲

### 基础知识

4. 既然你比较熟悉web开发，那介绍一下http协议吧。

> 回答的时候感觉有点懵，说了一下http请求有请求头和请求体，介绍了常用的请求方法（Get/Post/Head）以及状态码。

5. 看到你简历里有写大创使用了RSA加密，那介绍一下https加密的实现方法？在第一步需要client向server发送自己支持的加密算法，如果client不支持任何加密算法，https的处理方式是什么？

> https的对称加密，非对称加密的具体密钥交换流程我有讲出来，但client不支持任何加密算法出现的情况我实在是不清楚，只能坦诚说自己不知道了。（后来查了查，应该会出现加密套件不匹配错误？）

## 反向提问

最后是我的提问环节，我主要问了两个问题：

1. 投后台开发简历时，[招聘网站](https://join.qq.com/)上只写了c/c++/java三门语言，如果我能够通过面试的话需要考虑转语言吗？

> 面试官答：腾讯内部有很多语言的岗位，他们是Golang部门，是招聘网站上文案的失误。

2. 投简历时我选择的开始时间是6.1，但后来发现有一门课程需要占用一些额外时间，关于开始实习的时间能否再协调？

> 面试官答：开始实习的时间不是很重要，应该可以再协商，重要的是实习多久。

## 总结

总体来讲我对这次面试是比较满意的，不是因为自己在面试中表现有多好（实际表现很差），而是说面试过程让我学到了很多。

一方面，这是我有生以来的第一次面试，而且上来就是腾讯这样的大厂，紧张是在所难免的。开始面试时，我甚至感觉自己学过的所有东西都忘得一干二净了（第一题没有做好也有这部分的原因）。所幸我遇到的是一位和蔼的面试官，看到我卡壳之后并不咄咄逼人，而是试图引导我往正确的方向去思考，一来二去也缓解了我的紧张情绪，让我在后续过程中有机会稳定发挥出自己的正常水平。

另一方面，通过这次面试，我也看到了自己与大厂之间的差距，趁现在还有时间可以有针对性地做准备。就我目前的感觉，腾讯的面试是更加偏向于底层的，这意味着如果我有机会参加二面的话，需要更加详细地了解Golang的底层数据结构、逃逸分析、调度器、垃圾回收、并发模型等内容。

最后，面试官的态度也给我留下了深刻的印象。本来面试官家里临时有事需要调整一下面试时间，我说自己最近都有空，可以多延长一些时日方便他先处理，但为了不耽误我的时间，他还是把时间约在了第二天的晚上。第二天面试结束后道别，我感谢的话还没有说出口，面试官却先对我说：“这次面了一个多小时，辛苦你了”，让我很是感动。

## 结果

第二天查询，发现进入到二面状态。

# 第一次复试（50分钟）

## 自我介绍

## 项目

1. 大创项目的设计、架构、创新点、改进点、应用价值。

> 这里有在尽力回答，但可能因为这个项目过于简单，而我说的又太含糊了，明显感觉面试官不满意。

2. Java论坛系统的数据库设计、难点、鉴权方式等。

> 大概说了有一个用户表、一个帖子表和一个帖子内容表。难点确实没啥，就没有说到。鉴权方式使用JWT。

3. 详细介绍一下JWT，除了JWT方式外，还有什么鉴权方式？JWT方式鉴权和Session-Cookie方式鉴权方式的对比及优缺点？前后端分离就不能使用Session-Cookie吗？

> 因为很久没有复习，只说到了JWT内可以存储部分数据（面试后查了查才想起来JWT的Token内有Header、Payload和Signature）。
> 除JWT外还有Session-Cookie方式。
> 
> 对比来说，因为JWT在服务器端是无状态的，因此更适用于服务器集群等分布式场景，Session-Cookie是传统方式，Cookie中存储有Session-id。
> 
> 我当时确实认为前后端分离不能使用Session-Cookie，于是就这样回答了。

## 其它

1. 你的博客是原创的吗？介绍一下《查找相同字符串》里使用了几种方式？

> 是，但那篇是半年前的了，有点想不起来。

2. 进程、线程和协程的区别？有没有使用过多线程编程？

> 前一问是八股文，正常回答就可。
> 后一问，答用过但使用不多。

3. 为什么会想要学Go语言？

> 个人比较喜欢了解新兴的事物，接触过rust和go，因为golang足够简单，于是进行了一些学习。后来有查过一些工作岗位，发现大厂一般都有golang部门，这门语言的发展前景比较广阔，于是有了专心Golang的想法。

1. 学校的实习安排。工作地点的问题（意向北京，但他们在深圳）。

> 介绍了开始实习的时间，表达了工作地点无所谓的想法。

## 反问

1. 面试我的工作部门？

> 腾讯的信息流部门。

2. 有没有什么给我的建议？

> 加强实践，了解后台开发的基本原理。

## 结果

两天半后，官网变灰，提示流程已结束（挂掉了）。

-----

一周后接到北京的电话，对方介绍说自己是北京腾讯，问我还有没有实习意愿，得到我的肯定答复后约了一下初试时间。

# 第二次初试（50分钟）

因为曾经已经过了一次初试，所以这次没有问基础知识，全都是Go语言相关问题。

## 自我介绍

## 口述

1. 你使用的Go版本？讲一下Go的版本变化。

> 因为我这里使用的是Arch Linux，经常会滚动更新，所以使用的一直是Go的最新版。
>
> 版本变化只说出了两个：Golang 1.11版本加入了go mod，1.16版本加入了静态资源的打包。

1. 有了解过Goroutine吗，介绍一下调度方式。

> 有了解过，介绍了一下GMP调度模型。
>
> 但面试官追问G阻塞M时P内的G会不会饿死，我答了是。（后来才想起来这时候M和P会放弃绑定，由其它的M执行该P）

1. Go语言sync包内的类型？

> 答到了sync.Map（并发安全的map），sync.Mutex（互斥锁），sync.RWMutex（读写锁），sync.WaitGroup（统计信号量）。

2. Mutex和RWMutex的区别？

> Mutex是不区分场景的普通锁，RWMutex是允许多读，只能单写的锁。

## 编程

1. 实现一个m个task，n个worker的模型。

> 主要考察channel和routine的用法。

2. 实现一个并发安全的map，实现Get/Set/Add方法。

> 考察RWMutex的用法。

## 反问

1. 自己的面试表现。

> 挺好的，可以发现我的Go语言比较熟练，但对更深入的一些东西了解不够。
>
> 推荐了我一本书：《Effective Go》。

1. 面试的部门职责。

> 和上次面试相同，也是信息流方面的。

# 第二次复试（24分钟）

## 自我介绍

## 语言

1. 是学校有开Go语言的课，还是自己自学的？为什么要想要自学Go？

> 答自学，原因同第一次复试。

2. Go与C++的区别？与Java的呢？

> Go和C++都是原生语言，首先肯定C++可以实现Go的所有功能，但Go的封装层级够高，使用也够简单。
> 
> Java作为运行在虚拟机上的语言，个人认为在性能上是比不过Go的，不过胜在历史悠久，有着成熟的软件库与项目管理工具，而Go的这方面要差一点。

## 基础

1. 计算机网络七层模型，TCP/UDP、ICMP、IP、路由器、交换机的层级。

> 由于计算机网络很久没看，居然忘记了网络层，而且最后路由器的层级也答错了。（实在不应该）

2. 有学过操作系统吗？进程与线程的区别？32位Linux机的进程ID范围？

> 操作系统这个学期正在学。进程与线程的区别是八股，后一问没有了解过，答不会。

3. SQL查询18到25岁的人，如何按照年龄排序？如何加快查询速度，如果不加索引，该语句的查询方式是怎样的？

> 前一问很简单。
> 
> 后一问答了建立索引。如果不加索引，查询方式是遍历。

## 项目

介绍一下你的项目？讲一下它的设计与难点？

## 反问

1. 我的表现如何，有什么建议吗？

> 语言只是工具，最主要的还是基础，应该加强对基础知识的掌握。

2. 实习时间相关问题。

> 最少要求三个月，具体的时间可以和HR聊。

## 结果

面完本来以为自己要挂了，正在和朋友诉苦，查了查发现自己居然过了。（感谢面试官！）

# HR面试（23分钟）

## 自我介绍

## 其它

1. 前面面试官反馈说你Go语言掌握比较好，为什么要学Go语言？
2. 你理想的工作？
3. 家庭情况，父母是做什么工作的？有兄弟姐妹吗？
4. 绩点排名，为什么绩点不理想？将来规划（保研/考研/就业）是什么？
5. 比起其他保研和考研的同学，你的优点是什么？
6. 介绍一个你的项目，主要讲一下项目的目的，功能和你的职责。
7. 大创是如何组队的，在大创中学到了什么？
8. 有无直系亲属在腾讯？实习的时间？
