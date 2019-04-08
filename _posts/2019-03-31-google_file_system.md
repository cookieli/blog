---
layout: post
title: "google file system"
comments: true
description: "blog"
keywords: "dummy content"
---
### Archtecture
#### single  Master
一个gfs中只有一个master，一个master的好处是在处理chunk的放置，replication的选择的时候可以依赖全局信息，比较简单，但是过多的与master交互会成为整个系统性能的瓶颈，因此client与gfs交互时，应尽量减少与mater的交互。实际交互过程如下:<br/>
client设置file和chunk信息，交给master,master返回相关metadata例如chunk handle，和chunk server以及replication,client存入cache中，并建立chunk以及相关信息的表，以供查阅,后续直接和相应chunkserver交流<br/>

#### chunk size
由于大文件的普遍性我们可以考虑大的chunk size，大的chunk size可以减少client 与master的交互，可以减少clienthe master要存储的相关信息，可以减少network IO.<br/>
其缺点在与如果一个文件较小，仅有一个chunk时，如果有多个client同时访问会造成hotspot<br/>
这个问题的严重性在批处理系统中得以体现:<br/>
一个可执行文件要在上百个机器上执行时，对这一个文件的访问会造成overload,此时就要加大该文件的replica,同时错开应用的开始时间，还可以采用从其他已经开始执行的应用中获取信息的能力<br/>
#### metadata
master 维护的数据结构有 
1. namespace of files and chunks <br/>
2. mapping fron file to chunk <br/>
3.  chunk 和 replica 的location<br/>
他们都被放在内存中，我们需要格外注意内存和数据的大小<br/>
##### chunk location
以heartbeat 让chunk server 和 master沟通，信息传递<br/>
##### operation log
operation log 可以保证metadata更新的可靠性，还可以保证在master crash后依旧保持一致性<br/>
operations log 维护重要metadata 变化的记录和并发操作的逻辑时间线<br/>
如何保护Log的可靠性和有效性:<br/>
1. 将log 分别存储在local disk 和remote disk上多份保存<br/>
2. client 发送请求后，master完成操作，将相关record log flush到local和remote，在response client<br/>
如何利用log 进行恢复:<br/>
 利用checkpoint 方式存储log,crash 后从最近的checkpoint开始恢复<br>
 checkpoint如何建立:<br/>
 1. checkpoint建立需要时间<br/>
 2. 建立checkpoint的同时不会影响新的mutation，开启新的线程<br/>
#### consistency model
第一， 要理解何为consistent 和defined，在mutation发生后，所有client看见的内容是一样的称为consistent，如果能知道mutation修改了什么就是defined.<br/>
##### Guarantees by GFS
mutation主要包括write和 record append，我们主要考虑record append。<br/>
record append要保证其原子性，要保证并发状态下至少有一个发生，要保证mutation开始的offset由gfd给定，并标记defined region 的开始，其间可以添加paddding或record dup， 并会被认定为inconsistent<br/>
在经过一系列mutation后，gfs可以保证修改区域是defined并包含上一个mutation修改的内容,如何做到的:<br/>
1. 在所有replica上采取同样顺序的Mutation<br/>
2. 去检测是否有过期的replica,利用版本号<br/>
cilent如何保证cache是有效的，而不只是过期数据:<br/>
1. cache 加上 timeout <br/>
2. cache 更新每当打开一个文件时<br/>
3. 由于数据都是append的，可以在检测异常后和master沟通<br/>
如何处理component failure:<br/>
1. checksum 检测<br/>
2. replica restore<br/>
3. 所有数据都corrupt返回error而不是data<br/>
##### implications for applications
applications如何容纳relaxed consisency model:<br/>
1. appends not overwrites<br/>
2. checkpointing<br/>
3. write self-validating, self-identifying record<br/>
application的读和写:<br/>
1. 写采用 append at least semantics
2. 读采用checksum 来进行确认是否是defined region，如果存在padding 或者record fragment，我们既可以选择忽略也可以选择过滤<br/>
### System interaction

#### Leases and mutation order
lease 维护跨副本的修改操作集合，顺序未定<br/>
master 将lease 授予 replica, replica称为primary，primary在lease中决定线性的mutation执行顺序<br/>
所有replica应用该顺序<br/>
具体交流细节:<br/>
1. lease 有时限，可以延长
2. lease可以撤回
3. lease可以给予非primary的replica，如果primary出问题的话<br/>
交流过程:<br/>
#### Data flow
如何更好使用高带宽低延迟的网络呢:<br/>
1. 采用 链式传输而非树式传输<br/>
2. 每次只传给最近的chunkserver<br/>
3. 采用tcp传输<br/>
#### Atomic record appends
record append 的操作机制: client 提供数据， gfs提供offset。
好处:<br/>
当存在许多client写操作时, 传统的写需要添加十分复杂的并发机制，而 append只相当于多生产者，单消费者队列<br/>
操作过程: <br/>
类似于data flow<br/>
如果one replica fail的情况下:<br/>
1. 告诉client重新尝试一次
2. client 联系master,然后重新尝试
其他replica要pad，直到具有相同的offset
最终结果是defined和inconsistent的内容相互交错，前面已经介绍过如何处理<br/>
#### snapshot
采用技术: copy on write<br/>
撤回lease或等其失效, 记录 操作log到disk，应用log，将相关metadata复制，指向chunk的reference + 1.<br/>
对chunk进行操作时，发现reference > 1, 本地磁盘开辟复制，进行操作然后reply<br/>
### Master operation
#### Namespace Management and Locking
GFS的文件系统(namespace):<br/>
1. 表示为一个lookup table将pathname和metadata映射起来，将pathname用prefix tree表示<br/>
2. namespace上每一个结点都有一个write-read lock.<br/>
#### Replica Placement
基础问题:<br/>
1. GFS cluster分布在不同的级别的，有属于同一个(机架)rack的,有分属于不同rack的，同一个rack的带宽高一点，进出rack的带宽低一点，不同rack的机器传递信息要跨过几个交换机<br/>
考虑特性:<br/>
reliability, availability, 最大带宽。不仅要将replica分布在不同机器上，还要将其分布在不同rack上<br/>
#### Creation, Re-replication, Rebalancing
create chunk时需要考虑的问题:<br/>
1. 放置在磁盘利用率较小的机器上<br/>
2. 每个机器recent creation有数目限制<br/>
3. 放置不同的chunk上<br/>
re-replication出现原因:<br/>
小于要求的数目<br/>
re-plication 要考虑优先级，具体执行过程类似于creation,限制带宽<br/>
rebalance: replica的分配和再分配, load-balancing<br/>
#### garbage collection
##### mechanism
采取lazy模式，删除一个file时，将file命名为特殊的名字，回收器遍历整个namespace，找到相关file，观测其是否存在超过一定时间，然后删除，顺带抹除metadata,会产生orphran chunk<br/>
对于chunk namespace一样过程，并通过heartbeat和chunkserver交换信息，chunkserver根据信息释放空间<br/>
##### discussion
在文件系统中垃圾回收较为容易实现, 因为相关chunk 和file容易被定位<br/>
优点:<br/>
1. simply and reliably，对于master未知的replica也可以清除
2. regularly backend
3. 更有效面对误删
缺点:
磁盘不够时会出问题，<br/>
补救:
当一个被删文件又被删除的时候加快回收进度<br/>
对namespace的不同地方加以不同的replica和reclaim策略<br/>
#### stale Replica detection
利用版本号及其更新进行操作<br/>

### Fault tolerance and Diagnoise
1. Fast recovery<br/>
2. chunk replication<br/>
3. master replication:
利用operation log恢复，如果machine坏了用monitor beyond gfs恢复，生成的shadow master只能提供read-only access to file system,其他类似与priamry<br/>
#### Data integrety
目的：让chunkserver独立的利用checksum解决数据完整性问题<br/>
由于checksum占比较小，读不占io<br/>
写得时候append要根据上一个checksum计算新的checksum，即使上一个crash了<br/>
overwrite必须要检测被占区域开始和结尾块<br/>
空闲时进行检测和替换<br/>

