+++
title = "DDIA 数据密集型系统设计"
weight = 1
order = 1
date = 2021-12-02
insert_anchor_links = "right"
process = 0.333
[taxonomies]
tags=["ddia","reading"]
+++
> 学习“原理性”的东西，才是永远不会过时的

<!-- more -->
关于DDIA的阅读笔记
## 第三章
在本章，ddia主要介绍了常见数据库的索引结构。数据的处理大致分为两类，分别是OLTP（On-Line-Transaction-Process,联机事务型)和OLAP(On-Line-Analyze-Process,联机分析型)，针对这两种数据处理的特性，需要对数据的存储结构有不同的优化。

### OLTP型
本小节主要介绍朴素日志型存储结构，LSMTree和Btree。以及它们的优缺点及适用场景。
### hash索引结构
#### 朴素日志型数据索引
采用这种索引结构的典型数据库有Bitcask,该索引结构是一种hash结构，即`<key,value>`形式，其中真实的3