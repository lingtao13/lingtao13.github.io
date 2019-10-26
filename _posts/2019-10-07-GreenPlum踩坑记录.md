---
layout:     post
title:      GreenPlum 踩坑记录
subtitle:   
date:       2019-10-07
author:     LT
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - GreenPlum
    - data warehouse
    
---

# 问题描述

使用copy from导入postgresql时，报ERROR:  invalid byte sequence for encoding "UTF8": 0x00错误

### 尝试解决
数据文件是我从oracle导入的，文件编码为utf-8。这个报错还会提示行数，由于我的文件特别大，vim打不开文件，于是我用sed命令把报错行数提出来，再用vim打开，发现并没有什么异常。我用split命令按行数切割后，部分文件也可以导入。

后来参考了postgresql官方文档发现，这是postgresql独有的错误信息，直接原因是varchar型的字段或变量不接受含有'\0'（也即数值0x00、UTF编码'\u0000'）的字符串 。官方给出的解决方法：事先去掉字符串中的'\0'。

### 解决办法
>用sed 命令替换掉 0x00 

	sed -i 's/\x00//g;' file
参数说明 -i 表示在原文件直接替换，s/表示替换，/g表示全局替换