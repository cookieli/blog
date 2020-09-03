## Codestitcher 阅读笔记

      目前cpu上的cache和TLB的容量每年的进步约等于无，但是代码的体积却逐渐爆炸， 代码的体积爆炸代表着unrelated code也逐渐增加，由此导致导致icahce 和iTLB的miss也增加，严重影响程序性能，因此，构造最优的代码的layout很重要。这篇论文主要解决的就是这个问题。
      
### 关于优化codelayout的背景知识
    目前业界主流的做法是[Pettis-Hansen Method](https://dl.acm.org/doi/10.1145/93548.93550) .