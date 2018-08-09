---
layout:     post
title:      比特币与加密货币技术
subtitle:   加密货币简介
date:       2018-08-08
author:     LT
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - cryptocurrency
    - cryptography
---


# Bitcoins and Cyptocurrency Technologies
###### reading notes by lingtao(LT) start from 2018-08-08 

## 第一章 密码学与加密货币简介
### Chapter 1 Introduction to Cryptography & Cryptocurrencies

货币（currency） 是需要一些方法去控制供应量和必需多种的安全属性去避免欺骗（控制货币发行量和类似真伪识别）

加密货币（cryptocurrencies）也需要使用安全措施阻止人们篡改系统状态(tamper with the state of the system),它的的特征之一是与法定货币不一样，它的安全规则只需要单单被技术所执行，并不需要依赖一个中心认证（去中心化）

> 加密货币对密码学的使用广而深，可以防止篡改系统等功能，在理解加密货币之前，我们需要先了解一些加密货币所依赖的密码学的基础知识。

### 1.1 哈希函数

> 特点很明显，哈希函数是一个数学意义上的函数，包含以下三个性质

1. 输入的数值可以是任意长度
2. 输出一个固定长度的结果，假定256bit。
3. 可以高效的计算，即给定输入，便可以在合理的时间内得到结果。

它有如下几个特性：

1. 碰撞抵抗(**collision-resistance**)：碰撞的意义是对于输入x != y 哈希值H(x) =H(y) ，所以碰撞抵抗就是说没有人能找到这样的x,y使得H(x) =H(y)。(只是计算条件有限的情况下，人力和时间成本找不到这样的）【应用：消息摘要：是一个加密散列函数，包含由单向散列公式创建的一串数字。 消息摘要旨在保护数据或媒体的完整性，以检测消息任何部分的更改和更改。】

2. 隐秘性(**hiding**):隐秘性的意义在于无法逆向推倒，是某种意义上的 **one-way-function**          or **trapdoor-function** ，攻击者无法从哈希之后的值反推出加密前的值。【应用：commitment】

3. 解题友好(**puzzle-friendliness**):对于任意





