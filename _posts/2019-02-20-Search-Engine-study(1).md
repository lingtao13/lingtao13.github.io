---
layout:     post
title:      《Search Engine study(1)》
subtitle:   
date:       2019-02-20
author:     LT
header-img: img/post-bg-map.jpg
catalog: true
tags:
    - 搜索引擎
    - 爬虫
    - 读书笔记
---
# Search Engine study (1)

> 前段时间的工作中，数据的采集与清洗工作已经大致完成了自动化流程，接下来需要对数据的产出以及数据的深度加工做准备，一来Elasticsearch作为搜索引擎可以很好的快速筛选几十亿数据，通过接口可以完成完成数据的对外服务。

> 首先我准备阅读张俊林的《这就是搜索引擎》作为入门，然后细读《Manageing Gigabyte》 这两本书作为作为学习搜索引擎的基础

首先进入这就是搜索引擎的学习中

### Chap.1 搜索引擎及其技术架构
-
##### 1.搜索引擎技术发展史
史前的一代：分类目录的一代 -> 第一代：文本检索的一代 -> 第二代：链接分析的一代 -> 第三代：用户中心的一代

##### 2.搜索引擎的3个目标
更快，更全，更准

##### 3.三个核心问题
用户的真正需求是什么

哪些信息是和用户需求真正相关的

哪些信息是用户可以信赖的

![search-engine.png](https://upload-images.jianshu.io/upload_images/7232713-02c4f7626d578a75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> <center>搜索引擎架构</center>

第一章总是习惯性扫过去，一般来说不算章节，仅作为仪式感

### Chap.2 网络爬虫

##### 1.通用爬虫框架
首先搜索引擎面临的问题就是：如何能设计出高效的下载系统，能将数以百亿计的海量网页下载传送到本地，在本地形成互联网网页的镜像备份。

网络爬虫即起此作用，是搜索引擎系统很关键也很基础的组件（ps:对于本地文件系统的搜索引擎的我来说此章作用不会特别大，会爬虫，有所涉猎即可）

![image.png](https://upload-images.jianshu.io/upload_images/7232713-3bfdc9e58c781d76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图所示的就是一个通用的爬虫框架流程

![image.png](https://upload-images.jianshu.io/upload_images/7232713-230520f49b450f55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是对所有网页的划分，可见暗网大概就是主流引擎的不可知网页的一部分

爬虫又分为三类：

1.**批量性爬虫**：有比较明确的抓取范围和目标，当爬虫达到这个目标后立即停止。

2.**增量型爬虫**：持续不断的抓取，且定期更新已经抓取的网页（已过期网页即是有些网页变更导致url,ip等地址发生改变。）通用的商业搜索引擎爬虫基本属于此类

3.**垂直型爬虫**：只关注特定的主题或内容，属于特定行业的网页，对于健康网站来说，只需要从互联网页面里找到与健康相关的页面内容即可，其他行业的内容不在考虑范围内。（大概我们项目的爬虫属于此类）

##### 2.优秀爬虫的性能

- 高性能：高效的爬取速度，高效的数据结构
- 可扩展性：易于增加爬取服务器和爬虫数量
- 健壮性：处理非正常状况的能力（编码不规范，被抓服务器突然宕机，爬虫服务器重启后断点续连）
- 友好性：保护网站的部分私密性，减少被爬取网站的网络负载（robot协议等）

##### 3.爬虫质量的评价标准
- 抓去网页覆盖率
- 抓取网页时新性
- 抓取网页重要性

做到以上三点，搜索引擎用户体验必佳

![image.png](https://upload-images.jianshu.io/upload_images/7232713-1746fa3700c518dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于资源有限，所以以上三个标准请可能地说清楚了爬虫系统为增强用户体验而奋斗的目标

举例：Google
至少包含了两套不同的爬虫系统，一套为fresh bot 主要针对时新性，一套为deep crawl boot，主要针对更新不是很频繁的网页，以天为单位更新。除此之外，Google 投入了很大尽力研发针对暗网的抓取系统。

![image.png](https://upload-images.jianshu.io/upload_images/7232713-f9e8bf0586008859.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 4.抓取策略

由于爬虫爬取的结构通常以树/图的结构分布，所以抓取策略的选择也比较重要

- 宽度优先遍历（breath first）
	- 都很熟悉吧 ᕙ(•̤᷆ ॒ ູ॒•̤᷇)ᕘ  
- 非完全PageRank（Partial PageRank）
	- PageRank：链接分析算法，衡量网页重要性
	- 先对所需page进行优先级排序，优先度越大越先下载
- OCIP策略（在线页面重要性计算，Online Page Importance Computation）
	- 改进的PageRank算法，将现金分发到下属页面
	- 现金越多的页面下载有限度最大
	- 无须迭代过程，计算速度远快于PageRank
- 大站优先策略（Larger Sites First）
	- 以网站为单位衡量网页重要性
	- 优先下载大型网站，流量更多更大的网站

![image.png](https://upload-images.jianshu.io/upload_images/7232713-77155ddeab39ff7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 5.网页更新策略
任务是决定何时重新抓取之前已经下载过的网页

- 历史参考策略
	- just follow the history behavior
- 用户体验策略
	- 搜索越多的网站更新越频繁
- 聚类抽样策略
	- 根据网页一些可以预测的属性进行更新


##### 6.暗网抓取
>暗网的数量比，可以与宇宙中的暗物质占比相似，大约百倍于明网（Surfacing Web）网页

为了增加信息覆盖率，暗网爬虫的意义就诞生了。目前大型搜索引擎服务供应商都将暗网挖掘作为重要研究方向，这直接关系到索引量的大小【Google 重点研发，百度“阿拉丁计划”】

- 查询组合问题
- 文本框填写问题

有兴趣的朋友可以自己多看看此章，在此一笔带过不再赘述

##### 7.分布式爬虫
>对于商业爬虫来说，分布式是必须采用的技术，这样才能在短时间内完成一轮抓取

![image.png](https://upload-images.jianshu.io/upload_images/7232713-34fc7e746f5c3d4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 主从式分布爬虫 （Master-Slave）
	- 不同服务器担任不同分工
		- URL分发
		- 实际网页下载
	- 负载均衡
- 对等式分布爬虫（Peer to Peer）
	- 分工相同，各自负责自己的url抓取及网页下载
	- URL分发模式
		- 哈希取模
		- 一致性哈希

### 总结：
- 从爬虫设计的角度讲，优秀的爬虫需要具备：高性能、高可扩展性、健壮性以及友好性
- 从用户体验的角度讲，对爬虫的工作效果评价标准：抓去网页覆盖率，时新性，重要性
- 抓取策略，网页更新策略，暗网抓取和分布式策略是爬虫系统至关重要的四个方面内容，决定了其质量和性能



