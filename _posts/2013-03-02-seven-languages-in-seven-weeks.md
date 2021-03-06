---
title: 《七周七语言》有感
author: Dozer
layout: post
permalink: /2013/03/seven-languages-in-seven-weeks.html
categories:
  - 大杂烩
tags:
  - DotNet
  - CSharp
  - Java
  - 并行
---

### 七周七语言

前一段时间看完了《七周七语言》，这本书的书写风格有点像教材，会要求你回家做作业。

但说实在的，不做作业真的无法理解里面的全部内容。

或许抽空我也应该再看一遍并且把里面的作业做一遍。

&nbsp;

之前对 C# 最熟悉，所以难免会把里面的各种编程模型和 C# 进行比较了。

<!--more-->

### 约束—逻辑编程(Prolog)

> Prolog出身于一个专用于约束-逻辑编程的语言家族。我们用Prolog编写各种应用,全是为 了解决一小类问题。然而,解决这一小类问题获得的成果却蔚为壮观。简单地说,这类问题为某 个已知问题域定义一些逻辑约束,然后就可以用Prolog求出这类问题的解。

当我看到 Prolog 这门语言的时候，实在是让我眼前一亮！简单的来说， Prolog 就是用来做<a href="http://baike.baidu.com/view/1636209.htm" target="_blank"><strong>规则引擎</strong></a>的。

前一段时间，部门的另一个项目组在 C# 中利用 Ruby 实现了规则引擎。本来对这块还不是很理解，但是看完 Prolog 的介绍后，对规则引擎的基本概念了解得更深入了。

还记得高中的时候想：老师是怎么排课表的呢？能不能用一个算法来排一个课表？

现在回想起来，如果老师们有 Prolog，那这一切是不是就太简单了？

用 Prolog 编程，其实不是在编程，而是告诉程序一条条规则，也就是逻辑约束。

Prolog 就会在内部利用相关的算法进行高效的运算，然后告诉你满足条件的结果。

或许以后再遇到这样的问题，就不要去编写复杂的算法了，交给规则引擎吧～

&nbsp;

### 函数式编程(Scala、Erlang、Clojure、Haskell)

> 你已经了解到,函数式编程语言通常比面向对象语言有更强的表达能力。用它们写的代码显 得比面向对象语言更短小精悍,因为它们用来编程的手段要丰富得多。

看到各种函数式编程语言，和 C# 比较后发现，C# 真的很好地把函数式编程融入了自身。Java 在这块差距就很大了，Java 8 中才刚刚引入了 lambda 表达式…

书中的 Scala 正式为了在 Java 中实现函数式编程而产生的，它运行在 JVM 之上，可以直接使用现有的 Java 资源。

最后不得不佩服 C#，在面向对象的强类型编程语言上，优雅地实现了函数式编程。

更佩服的是，C# 在函数式编程的基础上，把它扩展为了 Linq，开发者可以使用同样的一套语法，实现两种功能（便捷地处理内存中的数据 & 便捷地处理远程数据）。

两种功能虽然在语法上完全一样，但在底层却截然不同（一个是 lambda 表达式，一个是表达式树）。

可惜 C# 出自微软之手，是一门商业化地语言，在开源社区不受待见。但如果抛开偏见，作者也应该把 C# 写入这本书～

&nbsp;

### 并发

并发是面向对象编程语言永远的痛。函数和数据捆绑在一起了，所以调用函数的时候就必须额外小心。

而在书中介绍的许多语言都是变量不可变、函数同样的输入就有同样的输出。这些规矩让这些语言在并发操作的时候变得格外简单了。

当然，它们也会有自己的局限性…

微软在 .net 的几个最新版本中，不断的改善着并发体验。书中介绍了几种并发模型，会让你有眼前一亮的感觉，但是目前我也没吃透，还需多读才能深刻地理解。

想想以前，一提到并发只知道 lock，真是井底之蛙啊！

&nbsp;

### 范型演进之路

> 如果你下定决心,以后多用函数式语言来编程,那么有几条路可供你选择,你既可以彻底与 OOP决裂,也可以选择较为温和的渐进策略。
> 
> 我们学这七门语言花了不少心力,但也见识到了叱咤风云四十余载的几门语言,以及多种编 程范型。但愿你现在已经对编程语言的演化有了一个正确认识。你可以看到三条截然不同的范型 演进之路。

在没有闭包、没有 lambda 地 Java 中，某些时候的开发效率真的是低了好多好多。

而 C# 在这条路上走得还是蛮快的。

2.0 泛型；3.0 lambda+扩展方法；3.5 Linq…

三大版本，几年的努力，最终对函数式编程有了一套优雅的实现方案。

4.0 并行；4.5 async+await 改善并行编写方式…

希望 C# 最终能有一套优雅的并行模型！

Java 君在这块你要加油了！
