---
layout: post
title: "condition variable"
comments: true
description: "If you have a Guest post.."
keywords: "dummy content"
author: GuestName
---
###要解决的问题
1. 一个thread会遇到检测一个condition来判断它是否需要执行的情况，，例如判断一个child是否完成，然后再继续执行的问题<br/>
解决方法一:<br/>
父，子线程共享一个变量，子线程完成后，将变量设为1，而父线程则一直检测该变量，变化后继续执行<br/>
缺点:<br/>
父线程空转并且会占用cpu资源，这是非常低效的，更好的方法是等待过程中父线程沉睡，<br/>
为此我们提供了condition variable，
### condition variable
condition variable是一个队列，一部分线程可在某些条件没有满足等待满足的过程中的时候将其放入队列中，另一些线程，可在改变这些条件的时候叫醒这部分线程，让其继续运行.<br/>
### the producer/consumer Problem
问题: producer线程生产数据并将其放入buffer中，consumer线程取数据并消耗数据<br/>
情况1: 1个producer, 1个consumer线程，代码:<br/>
![](https://raw.githubusercontent.com/cookieli/image/master/distributed_system/Screenshot%20from%202019-03-23%2020-05-48.png) 

问题， 当出现一个producer和2个consumer时，p1,c1,c2:<br/>
c1发现count = 0时,release lock，沉睡，p1运行，拿到lock，发现buffer是空的，放置，然后唤醒p1，注意此时p1未运行，还未接受锁， 然后p1释放锁,切换到c2,c2接受信息，执行完毕，此时切换到c1,c1继续执行, boom!!!!<br/>
解决方法，将if替换为while<br/>
这样做的遗留问题:<br/>
c1, c2运行先后沉睡，p1运行，放置，唤醒c1，执行完毕,p1继续运行，沉睡, c1运行，取出，唤醒放入condition variable的queue的线程，有俩条，如果唤醒c2的话，继续执行所有线程都会沉睡.<br/>
解决方式:用两个condition variable empty, fill, empty, empty用来叫醒producer, fill用来叫醒consumer<br/>
![](https://raw.githubusercontent.com/cookieli/image/master/distributed_system/two_condition_value.png) 