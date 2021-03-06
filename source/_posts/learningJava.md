---
title: 从零开始的Java学习
date: 2020-07-29 22:00:59
tags: [java,study]
categories: 学习笔记
photos: /img/banner/images/8.jpg
description: java学习笔记
---

## 背景

马上就要大三了。从我们这届开始，学院在大三进行分方向培养，我随个人兴趣选了软件开发，于是Java成为了我的必修课……

7月24日夏季学期结束之后，秉持着“反正闲着也是闲着不如找点事干”的基本原则，我开始了Java的预习（~~实则是摸鱼~~）。现在已经五天过去了，准备写个小总结。

<!-- more -->

## 教程的选择

这个问题我有请教过`NEU LUG`群的群友，得到的建议是看慕课，但考虑了自己的实际情况，最终我还是选择了适合自己的学习方式——读文档。在这里我选用的是[廖雪峰教程](https://www.liaoxuefeng.com/wiki/1252599548343744)。至于原因，主要是自己的`Git`/`JavaScript`/`Python`教程看的都是他的，自然`Java`也优先选用了。🤣

现在学习了一周，感觉教程各方面讲的都蛮细致深入，对`Java 14`的新特性有部分涉及，小节后常设练习题让大家巩固知识，有什么知识点不明白也可以评论请教，总体来看是一个很棒的教程。

## 当前进度

目前学习了教程前六章和第七章的部分内容，这些章节分别是：

+ Java快速入门
+ 面向对象编程
+ 异常处理
+ 反射
+ 注解
+ 泛型
+ 集合

## 学习体会

1. Java快速入门

   这一章讲解了Java的变量类型和基本语法，总体来看和`c语言`相似度很高，比较容易理解，但还是有一些需要注意的地方。例如：

   + Java12引入的switch表达式

     ```Java
     String fruit = "apple";
     // opt = 1
     int opt = switch(fruit){
             case "apple"->1;
             case "pear" ->2;
     }
     ```

   + Java的数据类型

     需要注意的是，除了整数、浮点数、字符和布尔类型外的所有类型，在Java中都是引用类型（对象）。

     Java内部使用UTF-16存储字符，因此Java的字符**占两个字节**。

     为了便于操作，每种基本类型都有其对应的包装类（如`int`的包装类为`Integer`）。

     String是不可变的，对String重新赋值，实质上是重新开辟了一片空间并赋值，并将String指针重新指向该片空间。

   + var关键字

     该关键字用于让编译器自动推断类型，类似于c++中的auto。

     ```java
     var s = "测试"; // s is String
     ```

   + 数组的遍历

     Java提供了类似Python`for ... in ...`的遍历方式：

     ```java
     String[] arr = new String[] { "1", "2", "3" };
     for (var item : arr) {
         System.out.println(item);
     }
     ```

2. 面向对象编程

   面向对象是老生常谈的概念了，有c++的学习基础，理解起来应该不会很困难。至于Java核心类，感觉更像是用到时再去查的东西，没必要熟练掌握。

   + 普通字段/方法和静态字段/方法

     就我个人理解，普通字段/方法是应用于对象（实例）的，而静态字段/方法是应用于类本身的。

     比如一个学生类，学生个体的姓名、年龄等应该是普通字段，而学生个数这种“类的性质”应作为静态字段。

     除此之外，静态方法还用于表示可以脱离实例本身而存在的功能，如：

     ```java
     // Number类的静态方法valueOf，用于构造Number实例，不依存于某个Number
     Number.valueOf();
     // Arrays类的静态方法sort，用于对数组进行排序，不依存于某个Arrays
     Arrays.sort();
     ```

   + final关键字

     final关键字主要有三个作用：

     + 用于修饰变量，表示变量不可更改
     + 用于修饰方法，表示方法不可被覆写
     + 用于修饰类，表示类不可被继承

   + 接口

     接口是抽象类的进一步抽象，只允许拥有公开的方法和静态且final的字段。

   + 面向对象基础的最后四节

     面向对象基础的最后四节：包、作用域、classpath和jar、模块阐明了Java代码的组织形式，个人感觉十分重要。刚开始看会有些懵，可以试着多读几遍（虽然我到现在也还很懵呜呜呜）。

3. 异常处理

   Java的异常处理采用`try...catch...`机制，可以将错误一层一层往上抛，该节内容较为简单，体会不多。

4. 反射

   反射是一种程序在运行期拿到对象信息的机制，可以在对实例一无所知的情况下调用其方法。例如：通过函数名的字符串调用函数。

   在这一章中我觉得较难理解的是动态代理小节（主要是不清楚有什么用），评论区有一位用户举了一个[很浅显的例子](https://www.liaoxuefeng.com/discuss/1279869501571105/1348125769859106)，让我受益匪浅。

5. 注解

   在我看来，注解是一个很有用的功能。教程中举了一个很实用的例子，即用自己定义的注解进行批量的参数检查。[这条评论](https://www.liaoxuefeng.com/discuss/1279869501571105/1337511613825057)很好地阐述了注解的作用。

6. 泛型

   泛型又是另一个难迈的坎，主要难点集中在后三节：擦拭法、extends通配符、super通配符。

   + 擦拭法

     Java的泛型实现使用擦拭法，即所有对泛型的处理工作都是编译器完成的，虚拟机对泛型一无所知。而这会导致很多问题。

     1. 泛型不能使用基本类型，因为泛型的实际类型是Object。
     2. 无法取得所带泛型的Class。
     3. 无法判断所带泛型的类型（instanceof）。
     4. 不能直接实例化T类型，需要借助额外的Class<T>。

   + extends和super通配符

     这两节主要讲在针对泛型的方法中使用extends/super通配符导致的性质差异。笼统来说就是，extends可读不可写，super可写不可读。

     要理解这个性质，需要明白向上转型和向下转型的区别：

     向上转型即通过父类引用子类，向下转型即通过子类引用父类。前者是安全的，后者是不不安全的。

     ```java
     class Person{}
     class Student extends Person{}
     Person person = new Student(); //√
     Student student = new Person(); //×
     ```

     因此，`<? extends Integer>`说明泛型内必然是`Integer`或其子类，在这种情况下，将泛型内的`Integer`或其子类赋给外部的`Integer`是向上转型，是安全的，而将外部的`Integer`赋值给内部的子类是向下转型，因此会出现错误。最终体现出来的性质为“可读不可写”。`<? super Integer>`与此相反。

7. 集合

   第七章我目前只学了将近一半，目前感觉较为简单。该章主要讲的是基本的数据结构，如数组、链表、哈希表等等的**使用方式**，因为不涉及到设计和实现，该章还是蛮轻松的。

   ----------

   > 未完待续...（ 随 机 更 新 ）😝