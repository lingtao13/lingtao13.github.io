---
layout:     post
title:      GreenPlum 学习（一）
subtitle:   
date:       2019-09-01
author:     LT
header-img: img/April-lies.jpg
catalog: true
tags:
    - GreenPlum
    - data warehouse
    
---

# GreenPlum 学习 （一）

GreenPlum 优点：

1.在数据量为TB级别是的性能表现非常优秀，单机性能比Hadoop快好几倍

2.是基于PostgreSQL的一个完善的数据库，在功能和语法上都比Hadoop上的SQL引擎 Hive 好用很多

3.有着完善的工具，相比Hive，体系完善，不需要花大精力改造，非常适合作为一些大型的data warehouse解决方案

4.可以与Hadoop进行结合，可直接把数据写到Hadoop上，还可以直接在数据库上写MapReduce任务，配置简单

部分观点及内容来自何勇、陈晓峰所著的《Greenplum企业应用实战》及其他博客，取其重点用作记忆及普及

##简介

###OLTP&OLAP

数据库系统一般分为两种类型：OLTP(On-Line Transaction Processing 联机事务处理)，也称之为生产系统，是事件驱动的，面向应用的。例如电子商务网站的交易系统就是典型的OLTP系统

以及OLAP(On-Line Analytical Processing 联机分析处理) 是基于数据仓库的信息分析处理过程，是数据仓库的用户接口部分。OLAP系统是跨部门的、面向主题的。

OLTP的特点有

- 数据在**系统中产生**
- 基于**交易**的处理系统
- 每次交易牵涉的**数据量很小**
- 对**响应时间**的要求非常高
- 用户**数量非常庞大**，主要是操作人员
- 数据库的各种操作主要基于**索引**进行

OLAP的特点有

- 本身**不产生数据**，其基础数据来源于生产系统中的操作数据（OperationData）
- 基于查询的**分析**系统
- 复杂查询常使用多表连接、全表扫描，牵扯的**数据量**往往非常庞大
- 响应时间于具体查询有很大关系
- 用户**数量相对较小**，其用户主要是业务人员与管理人员
- 业务问题不固定，数据库的操作**不能完全基于索引**进行

Greenplum 属于后者 OLAP 联机分析处理型

### GP与PostgreSQL的关系

#### PostgreSQL 

PostgreSQL是一种非常先进的对象-关系型数据库管理系统,是目前最强大，特性最丰富的和技术最先进的自由软件数据库系统之一，某些特性甚至连商业数据库都不具备。可以说是最先进的开源软件数据库，支持绝大部分主流数据库特性。

主要体现如下

- 函数/存储过程
- 索引
- 触发器
- 并发管理（MVCC）
- 规则（RULE）
- 数据类型
- 用户定义对象
- 继承
- 其他特性与扩展

更加详细内容请查阅资料

#### Greenplum

GP就是一个与Oracle、DB2、PostgreSQL一样面向对象的关系型数据库，可以通过标准SQL对GP中的数据进行访问存取

本质上是一个 **关系型数据库集群** 

由数个独立的数据库服务组合成的逻辑数据库

与Oracle RAC 的架构 **Shared-Everything**不同

Greenplum 采用 **Shared-Nothing** 架构

- **Shared-Everything**:一般是针对单个主机，完全透明共享CPU/MEMORY/IO，并行处理能力是最差的，典型的代表SQLServer
- **Shared-Disk**：各个处理单元使用自己的私有 CPU和Memory，共享磁盘系统。典型的代表 Oracle Rac,它是数据共享，可通过增加节点来提高并行处理的能力，扩展能力较好。其类似于SMP（对称多处理）模式，但是当存储器接口达到饱和的时候，增加节点并不能获得更高的性能 
- **Shared-Nothing**:各个处理单元都有自己私有的CPU/内存/硬盘等，不存在共享资源，类似于MPP（大规模并行处理）模式，各处理单元之间通过协议通信，并行处理和扩展能力更好。典型代表DB2 DPF和 Hadoop ，各节点相互独立，各自处理自己的数据，处理后的结果可能向上层汇总或在节点间流转。
我们常说的 Sharding 其实就是Share Nothing，它是把某个表从物理存储上被水平分割，并分配给多台服务器（或多个实例），每台服务器可以独立工作，具备共同的schema，比如MySQL Proxy和Google的各种架构，只需增加服务器数就可以增加处理能力和容量。

## 特性及应用场景

### 特性

- 支持海量数据存储及存储
- **高性价比**
- 支持 **Just In Time BI**
- 系统易用性（与PostgreSQL基本一致）
- 支持线性扩展
- 并发支持及高可用性支持
- 支持 MapReduce
- 数据库内部压缩

### 应用场景

- 查询速度快
- 数据装载速度快
- 批量DML处理快

->

- 面向应用的分析，构建企业级ODS/EDW、数据集市等

简单地介绍了Greenplum的背景，对比了OLTP、OLAP、Shared-Everything、Shared-Nothing等特性及优劣