---
layout: post
title: "context-sensitive analysis 和 type system 简介 "
comments: true
description: "too tierd"
keywords: "database"
author: lzx
---
### Chapter Overview
我们需要考虑每个语句在实际上下文中是否有意义的，对应有俩种技术: <br/>
1. attribute grammmars<br/>
2. Ad Hoc<br/>
### context-sensitive analysis
我们需要知道一些除了基本句意以外的信息,理解输入程序中的计算模型，值得表示，存储与流动，程序如何与外部设备交流。这需要deeper analysis即context-sensitive analysis<br/>
用来对代码抽象的方法有很多，我们主要讨论type system<br/>
####overview
在这一部分中，我们需要辨析contet-free和context-sensitive的不同:<br/>
首先考虑在生成可执行代码时我们需要解决的问题:<br/>
对于一个变量x来说:<br/>
1. 他的值的类型<br/>
2. 值的大小<br/>
3. 值应该被维护的时间<br/>
对于一个函数来说:<br/>
1. 其参数的多少，类型，以及去哪里去找我们需要的参数<br/>
2. 返回值的大小，类型，和去哪里找<br/>
还有变量的空间开辟问题<br/>
以上提出的问题相当一部分可以通过强类型语言中的类型声明来做，弱类型则要通过推导解决<br/>
以上问题大部分都超出了context-free的能力界限。context-free只能理解其syntax,无法理解其meaning。<br/>
syntax和meaning的区别在书p164<br/>
### An introduction To type systems
type是和数据值相关的属性的集合，该集合被所有该类型的值共有。<br/>
type system是一系列type的集合和规则来指定程序行为的集合。
#### The purpose of Type system
type system能够描述确定性语言的形式和行为。可以用来完成以下目的:safety, expressiveness, runtime efficiency<br/>

##### Ensuring Runtime safety
type system 可以辨别有问题的程序在他们执行之前。(不可能消除所有错误)<br/>
完成任务的手段:<br/>
1. 对表达式进行类型推断<br/>
2. 检查操作符对应的操作数是否满足规则<br/>
强类型语言完成infering和checking，而弱类型语言虽然增加了灵活性，但需要实现做大量的运行时推断和检测<br/>
##### Improving Expressiveness
可以更好的使程序行为简洁，例如operator overloading，不同的符号面对不同的类型有不同的含义, context free 做不到。<br/>
##### Generating Better Code
#### Components of a Type System
1. base type<br/>
2. compound and constructed types <br/>
3. type Equibalence<br/>
4. Inference Rules<br/>
书上相关内容对inference有一定补充<br/>

