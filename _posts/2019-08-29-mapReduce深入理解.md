---
layout: post
title: "A deeper understanding for mapReduce"
comments: true
description: "To learn mapReduce  twice"
keywords: "dummy content"
author: lzx
---
问题: 对巨大的数据集进行长时间的计算
总体目标: mapreduce可以让程序员更轻松的划分数据，使其在不同的计算机上进行有效的处理.
具体过程：程序员定义map和reduce函数，mp将其分布在数千台机器上，并且让其有效的处理巨大的数据。隐藏分布式的具体细节
### MapReduce的abstract view:
#### input被切分成m个文件
#### MR调用map function 对每个输入进行处理,生成一系列键值对，这是intermediate data.

#### MR系统执行group and shuffle, 将属于同一个key的的键值对聚合起来，构成一个list,将其作为参数传递给reuduce call.
#### mr调用reduce函数，对属于同一个键值对的list进行处理，得到一个聚合的结果

用更简单的方式描述是:<br/>
map(k1, v1) -> list(k2, v2) -> reduce(k2, list(v2)) -> k2, v3
### 具体实现细节
有一个程序master负责给空闲的worker分配任务，map任务或者reduce任务.,一个任务执行完毕后在执行新的任务。在map没有完成前不执行reduce,共有m个reduce task 和r个map task<br/>
在map过程中, library将input 解析来供map处理，生成中间数据，即一系列键值对，将其缓存在memory中，并定期的将其存在local disk中，利用partition函数将在磁盘中的存储分为9个部分，worker将其地址传递给master以便于rerduce后续的工作。<br/>
map执行完毕后开始执行reduce, master notify redcue worker, reduce worker 使用rpc访问存储中间数据的磁盘，读取所有数据，并且排序，然后reudce worker对已排序的中间数据进行iterate,并将key和对应value的list传递给user定义的reduce函数，将其结果输入到最终的文件中。<br/>
### Mapreduce scale well:<br/>
N台机器带来n倍的throughput
假设M, R远大于n
map 和reduce可以并行执行，因为他么是函数式的，只和输入输出有关，没有状态
map和reduce的唯一交互是shuffle.
### 如何减少slow network的影响
map input 来自local disk,gfs的copy,不经由网络，map的结果也存在local disk中，不经由网络。
中间数据切分成不同的key
### load balancing:
对于scalbility 至关重要的: n-1台机器等待最后一台完成任务
解决方法:让M和R的数量远大于n, 这样当某台机器上task执行完毕后，可以分配新的任务，这样task的长度就很可能不会制约完成时间
### fault tolerance
当一个server crash的时候，MR应该怎么办，
MR 只是简单的re run 失败的map和reduce.MR要求map和reduce都是pure function,这样函数没有状态，不操作除了对应的文件的其他的文件，没有communication, 只和输入有关，这样re run的结果依然正确。
