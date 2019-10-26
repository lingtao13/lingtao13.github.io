---
layout:     post
title:      GreenPlum 学习（二）
subtitle:   
date:       2019-09-01
author:     LT
header-img: img/home-bg-universe.jpg
catalog: true
tags:
    - GreenPlum
    - data warehouse
    
---

# GreenPlum 学习 （二）

针对系统 I/O 调度算法，做如下解释。系统 I/O 调度算法有四种，CFQ(Complete Fairness Queueing，完全公平排队 I/O 调度程序)、NOOP(No Operation，电梯式调度程序)、Deadline（截止时间调度程序）、AS(Anticipatory，预料 I/O 调度程序)。

下面对上述几种调度算法做简单地介绍。

CFQ 为每个进程/线程，单独创建一个队列来管理该进程所产生的请求，也就是说每个进程一个队列，各队列之间的调度使用时间片来调度，以此来保证每个进程都能被很好的分配到 I/O 带宽，I/O 调度器每次执行一个进程的 4 次请求。

NOOP 实现了一个简单的 FIFO 队列，它像电梯的工作主法一样对 I/O 请求进行组织，当有一个新的请求到来时，它将请求合并到最近的请求之后，以此来保证请求同一介质。

Deadline 确保了在一个截止时间内服务请求，这个截止时间是可调整的，而默认读期限短于写期限，这样就防止了写操作因为不能被读取而饿死的现象。

AS 本质上与 Deadline 一样，但在最后一次读操作后，要等待 6ms，才能继续进行对其它 I/O 请求进行调度。可以从应用程序中预订一个新的读请求，改进读操作的执行，但以一些写操作为代价。它会在每个 6ms 中插入新的 I/O 操作，而会将一些小写入流合并成一个大写入流，用写入延时换取最大的写入吞吐量。

实际上，虽然NOOP很快，但是由于会阻塞query 我们通常还是用deadline

