###Overview
DBMS的数据模型是record的集合，也可以抽象为file的集合，file由page组成。<br/>
file organization指的是如何安排file上的record，当file存储在磁盘上时，我们可以以一定顺序去排以便于检索，但这样做的代价十分昂贵，同时由于操作的多样性，一种排法无法满足所有需求。<br/>
我们引入了indexing的技术来辅助我们进行管理。<br/>
###外部磁盘的内存
磁盘和内存通过页为单元交换信息。磁盘io成为主要负担，优化io是重要任务。我们注意以下几点:<br/>
1. 连续访问代价远小于随机访问。<br/>
2. 可通过rid来找对应record存放的page。<br/>

file and access layer通过buffer manager访问磁盘<br/>
磁盘的空间通过disk access manager来访问.<br/>
###文件组织和索引
最简单的文件组织是unordered file或者heap file。
index用来组织磁盘上的data records来优化特定类型的retrieval operations。<br/>
我们可以利用index的search key来满足检索，我们还可以定义额外的index来满足不同的检索需要。<br/>
