---
layout: post
title: "Database basic intro "
comments: true
description: "Now I learn a cource mit 6.830 about database.And it'a a intro "
keywords: "database"
author: lzx
---

  我在学习mit的一门有关数据库的课程。这门课有6个lab和1个project，我正在努力的学习。目前已完成第一个lab。这部分博客将是我对课程学习的总结和lab的学习和教训。
   第一个lab引入三个知识点：1 . table是如何构建的  2. table以怎样的形式存储在磁盘和缓存的。3. 当一个operator发生操作时: 数据库的相关数据的读取过程是怎样进行的。
因为我的java 和建构一个大型的代码的能力不足。经常有冗余设计和过度设计的问题，因此，我在总结相关经验时会格外注重mit的代码是怎么组织和实现的，以及自己在其中所犯的一些低级错误。水平有限，请见谅。
### 数据库的基本构成
一个关系型数据库的基本构成是表(table)。而表由一个个tuple组成。每个tuple的中每一列的值代表对应attribute的值。

因而在lab1中，我们用俩个类来表述相应关系:tupleDesc 和tuple。tupleDesc确定的每一个tuple的范式，它规定了一个tuple的一系列Field,包括类型(type)和值(field)实现。Type采用了enum的实现方式，定义了getlen和parse获取的方法。field采用interface继承实现的方法去实现。field接口中定义了比较获取类型的方法。我对于接口的使用十分拙劣，所以强调一下。

而tuple类的实现包含了范式和对应值(field)的属性。此处，我们完成了一个table的基础单元tuple的实现。
### 数据库的存储结构

1. Catalog:
catalog 用于记录一个数据库中已有的表的信息，当我们向数据库中添加表的时候，先在catalog中注册，读取时也要先查询catalog。通过tableName, tableId, t来和table建立关系

2. BufferPool
显然，当有一个query时的时候我们不会直接访问磁盘，而是通过建立cache的方式去做(操作系统类似)，我们称之为bufferPool，bufferPool中以page为基础单元存储信息。我们限制bufferPool的大小并实现其读写基本操作。

3. HeapPage
上面谈到了基础存储单元page，下面讨论page的组成。page都有一个固定的大小，同时page具有独一无二的pageId,在lab1中，page由俩部分组成，header和存储的tuple，header的每一个bit代表对应tuple的slot。0 代表unintialised,1代表intialised。lab1中，一个page 只代表一个table的一部分。
现在我们讨论具体类的实现:
首先page的interface中要暴露获取id和content的方式。
在heapPage具体类的实现中,我们需要复原他在磁盘上的实际结构，除了实现借口的部分以外，我们通过byteArray的方式实现header，同时也要记录其schema,即tupleDesc。
要注意的一些细节:

                          1.   要指定page需要指定其tableId 和pageNum,通过PageId类来封装
                          2.   指定特定的tuple同理,通过recordId来封装
                          
4. HeapFile:
  类的实现中只是有tupleDesc和对应磁盘文件的属性，暴露的方法除了一些例如page的数目的基本方法的话，要注意他在读取page时是random access的以及这个类是直接操作磁盘的，不经过bufferPool。
 
 5. 代码细节的总结：
    代码在实现dbFileIterator的时候没有采用java提供的接口，而是自己定义了一个新的接口,暴露了open(), close(). rewind()的方法，开阔眼界。
    自己在写代码的时候，在处理ceil(a/b)操作的时候，a,b为整形而没有进行类型转换，丢失精度，debug了很久。引以为戒。
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
