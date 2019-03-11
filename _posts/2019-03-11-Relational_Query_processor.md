---
layout: post
title: "Relational Query processor"
comments: true
description: "to isssue Relational Query processor"
keywords: "query processor"
author: Lzx
---

### 一个relational query processor 的工作流程
1. 获取相关的sql语句(declarative sql statement) <br/>
2. 确认他，将其优化为过程间数据流执行计划（procedural dataflow execution plan）并执行<br/>
3. 获取数据，通常一次一个或者小批量<br/>
总结: <br/>
通常　query processing 被看做single-use, single-threaded task.
### Query parsing and authorization
#### parser的主要任务
1. 检查　query是否正确<br/>
2. 解析名字和引用<br/>
3. 将query变为中间形式(internal format) <br/>
4. 确定使用者是否授权(authorization) <br/>
#### 规范化表名
首先，parser考虑来自FROM语句的表的引用。他将表的名字规范化为完全确定的格式: server.database.schema.table。可以根据具体情况省略server，database。别名(alias> 也要规范化<br/>
规范化表名的作用是为了满足上下文相关的默认规则:允许单部件表名在规范中被使用。
#### 调用catalog
query processor调用catalog manager 来检查表是否已经注册到system catalog。它同时也缓存表的metadata在internal query data structures。<br/>
基于表的信息，它使用catalog 来确保属性的引用是正确的。属性的数据类型用来保证重载函数，表达式，操作符，常数计算的规则统一性。<br/>
#### 授权检测
授权检测完成使用者有权限对在query过程中的表，函数，一些对象进行操作。在对statement进行parse的过程中进行完全的授权检测并不总是可行的。我们会将一部分的安全检查延迟在执行过程中。<br/>
延迟安全检查的query　plan可以和使用者共享，并不会因为安全条件的变化而重新编译。<br/>
一些限制性检查，　例如赋值过程中类型不匹配有时也会被延迟到执行期间去做<br/>
### Query rewrite
query rewrite用来简化和标准化query。他利用query和存储在catalog中的metadata进行操作，并不允许访问数据。<br/>
query rewrite并不直接对query进行操作，而是对已经parse后的internal format进行操作，query rewrite后的格式与输入前的（internal format）保持一致。<br/>
Query rewrite的主要职责:
####1. view expansion(视图扩张)
对于from中出现的视图，rewriter找到catalog 中对应的视图定义，然后重写query,具体流程如下: <br/>
1. 用视图(view)所引用的表和谓词代替视图<br/>
2. 用view中表的列引用代替对view的引用<br/>
以上过程递归执行直到表中没有列引用。
####2. 简化常数计算
####3. 对谓词的逻辑重写
对于where语句中的谓词和常数进行逻辑重写，例如可将 not > 变为 <=等。<br/>
这种重写可用来提高匹配度和access methods的能力。<br/>
还可以用来消除不可满足的请求。<br/>
同时还能利用谓词的传递性推导出新的谓词。例如 x < 10 and x ==y 可以推导出 y < 10
####4. 语义优化
重要的一个应用是redundant join elimination.
####5. Subquery flatting and heuristic rewrites
大部分optimizer都只对隔离的单一的 select-from-where进行优化，不进行跨block优化，所以需要我们将query重写为更适合optimizer的形式，称为query的正则化。另一个比较重要的模块是将启发式的嵌套的query展开来，来形成single-block query，更好的为optimiser服务。
### Query Optimizer
query optimizer的工作是将query internal represitation 转换为高效的query plan来执行query.<br/>
query plan可以看做一个数据流图来通过query operator 图对数据进行流水线处理。
resulting query plan 可以被以多种方式表示
####A note on Query compilation and recomplication
SQL支持准备一个query, parse, rewrite, optimize,将生成的query plan存储起来，在后续的执行指令中使用它。这甚至可以应用于动态query中，在一个constant query 中有一个 program variable。这样做唯一的缺点是如果optimizer在代替该variable的过程中采用了不好的典型值，会导致生成差劲的query plan。<br/>
Query preparation 可用于形成常规的，批发的，数据预测性比较高的的query中，会极大提高效率。<br/>
动态构造sql中会更倾向于使用 query plan cache.<br/>
### Query Executor
query executor 操作于完全特定的query plan。这是一个连接operator的数据流图，其中operator封装了数据的访问和不同的query执行算法。数据流图有俩种表示方法，<br/>
1. 被optimizer编译成底层的op-code，相当于运行时的解释器。<br/>
2. executer接收到数据流图的表示，根据图的层次结构递归调用程序。<br/>
许多现代的query executor使用iterator model。每个iterator的输入代表了数据流图的一条边，所有query plan的operator，即数据流图的对应节点，都是iterator的子类。<br/>
典型的子类有filescan，indexscan，sort, join，duplicate-elimination，和grouped-aggregation.<br/>
iterator model的重要特性在于每一个iterator的子类都可以作为其他类的输入。这样，每个iterator的逻辑独立于父子，并且不需要特殊的代码做iterator的合并。（postgre sql) <br/>
#### Iterator Discussion
Iterator 的一个重要特性是他能够 couple dataflow with control flow.<br/>
A single thread can execute an entire query graph.<br/>
is quite efficient for single system(non cluster) query execution.
对于 parallel query execution，我们可以将并发和网络传输通过特殊的exchange iterator 封装，无需改变iterator model和query execution arch 就可支持。
#### Where's the data
每一个iterator 预先分配了一个固定数目的tuple descriptor，一部分用来输入，一部分用来输出，tuple descriptor 由一个列引用的数组构成，每一个列引用由一个指向内存tuple的引用和对应tuple的column offset构成。那么被引用的tuple如何在内存中实际存储呢？

##### BP-tuple
第一类是存储在buffer pool上页中的tuple。如果一个iterator构造tuple descritor 引用BP-tuple,它必须增加相关page的pin count.清除 tuple descriptor 则减少pin count。
##### M-tuple
第二类一种iterator implementation 可以在堆上开辟tuple的空间来放置tuple，通常通过复制bufer pool的 column或计算表达式来实现。
##### general approach
通常的实现方式是将buffer pool 中的tuple复制到m-tuple中<br/>
这种设计采用m-tuple作为唯一的tuple实现，简化执行代码的实现，避免pin 和unpin的bug。但是内存 copy的操作会成为瓶颈。
最好的操作是既使用BP-tuple又使用M-tuple
#### Data modification Statements
单独的读和写很容易，但是将读和写结合起来对同一组数据进行操作很难。<br/>
举例:

    UPDATE EMP
      SET salary = salary *1.1
      WHERE salary < 200000
      
   这段语句本意是找到小于200000的员工，加一次工资，但是有可能导致已经被update的员工满足条件再次update。
   
   解决该问题的技术
##### 由query optimizer选择能避免已更新column的plan。
##### 使用batch read-then-write tech
该技术 插入 RECORD-ID materialization和fetching operators 在index scan和data modification之间，
新操作的意义：
将待修改的tuple的RID存储在文件中，扫描文件获得实体ID,将获得的tuple放入 data modification之中。
### Access Methods
Access Methods 是控制对基于磁盘的数据结构的访问的程序。他支持unordered files(heap)和各种目录结构<br/>
Access Methods 提供的API是iterator API.<br/>
Access Methods 的init()方法接受search arg参数SARG，形式为col operator, constant, get_next返回满足参数的tuple。
向access methods传入sarg参数的意义:
1. 利用目录访问会更有效<br/>
2. sarg能保持干净的结构边界在存储引擎和关系引擎之间，并能获得良好的性能。<br/>
对2的解释：
如果sarg的检查由access methods的调用者完成 完成，那么对于access methods返回的tuple，他必须去做pin或unpin，复制或删除（m-tuple），这将消耗大量的cpu cycle，并有很多无意义的操作，对cpu 造成很大负荷，而交给access methods层去做的话则不需要。
B+-tree作为基础存储结构并且能够有效的处理row移动


  



