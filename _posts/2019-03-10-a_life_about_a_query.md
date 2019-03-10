---
layout: post
title: "一个query的一生"
comments: true
description: "to introduce a database by walk through follow the query."
keywords: "dummy content"
author: Lzx
---
   　　首先我们来看一个database的完整结构：
 　　![image](https://raw.githubusercontent.com/cookieli/image/master/database/database.png)
 　　
---
### 应用场景以及工作流程
#### 应用场景
机长请求本次飞机的乘客名单
#### 工作流程
1. 电脑调用相关api通过网络与　Client Communication Manager建立连接。
　　建立连接的方式:<br/>
　　		1. 	client 直接与　database server 建立连接　称为two-tier或者client-server模式<br/>
　　		2. 	通过middle-tier-server来通过某种协议来代理client和server的连接，称为three-tier模式<br/>
　　		3. 	还可以在middle-tier-server和DBMS直接加一层application server来代理连接。<br/>
　　连接方式的多样性对client communication manager的要求:   需要兼容多种连接协议。<br/>
　　Client Communication Manager的作用:<br/>
　　建立和记录来自请求者的连接，<br/>
　　回应请求者的sql命令   <br/>
　　返回合适的控制和数据信息。<br/>
　　
2. 收到sql的命令后，DBMS分配一个计算线程给该command，
     同时确保线程的数据和命令输出通过client communication manager与client连接。
     这些工作是由process manager完成。
     在此过程中最重要的任务是admmision control。借此决定系统立即执行请求或者选择延迟。
　
　
3. 完成query的资源分配和授权后，可以进行query的执行。该步骤通过relational query processor来执行。
   这部分模块先检查user的授权信息，然后将query的文本编译成internal query plan,编译完成后交由
   plan executor来执行。plan executor里集成了operator，这些operator都是具体关系算法的实现，
   用来实现例如join等方法。
   
4. 在query plan的底层，一些operator从数据库中请求数据，他们通过调用 transactional storage manager来完成任务。
该组件用来完成所有的数据访问和数据操作的任务。这个存储系统拥有用来控制和访问磁盘数据的算法和数据机构。(access methods)，
例如table 和　indexes。他还有buffer manager来决定 buffer和disk的数据传输。
回到请求query的过程，在利用access methods 访问数据的时候，query先要调用transaction manage code来保证transaction的acid属性。
在访问之前，先要向lock manager请求锁来保证query并发执行的正确性。当query更新数据库的时候与log manager交互来保证commit的
时候系统是durable的，否则undone.

5. 现在可以进行访问数据和计算结果的过程。整个过程是一个清空栈的过程。
  access methods 返回控制给plan executor operator，来处理计算结果，生成tuple后通过communication manager 来将
  结果返回给请求者，大量数据请求者通过多次请求来获取数据。然后　数据库进行资源的释放。
  
6. 未涉及到的模块: Shared Components and Utilities。catalog manager 用来authentiation(认证,), paring(解析)，和optimizing(优化)。memory manager用来开辟和回收内存。
 　
   
