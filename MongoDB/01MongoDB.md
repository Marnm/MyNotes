---
title: 01MongoDB
tags: []
---

# MongoDB介绍

Mongodb 由 C++语言编写的，是一个基于分布式文件存储的开源数据库系统。，是专为可扩展性，高性能和高可用性而设计的数据库， 是非关系型数据库中功能最丰富，最像关系型数据库的，它支持的数据结构非常散，是类似 json 的 bjson 格式，因此可以存储比较复杂的数据类型。

MongoDB 的（来自于英文单词“了 Humongous”，中文含义为“庞大”）是可以应用于各种规模的企业，各个行业以及各类应用程序的开源数据库。作为一个适用于敏捷开发的数据库，MongoDB的数据模式可以随着应用程序的发展而灵活地更新。
MongoDB 以一种叫做 BSON（二进制 JSON）的存储形式将数据作为文档存储。具有相似结构的文档通常被整理成集合。可以把这些集合看成类似于关系数据库中的表： 文档和行相似， 字段和列相似。

> MongoDB是专为可扩展性，高性能和高可用性而设计的数据库。它可以从单服务器部署扩展到大型、复杂的多数据中心架构。利用内存计算的优势，MongoDB能够提供高性能的数据读写操作。 MongoDB的本地复制和自动故障转移功能使您的应用程序具有企业级的可靠性和操作灵活性

## 2.1 MongoDB 数据格式

### 2.1.1 JSON
JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。JSON 采用完全独立于语言的文本格式，但是也使用了类似于 C 语言家族的习惯（包括 C、 C++、 C#、 Java、JavaScript、 Perl、Python 等）。这些特性使 JSON 成为理想的数据交换语言.易于人阅读和编写，同时也易于机器解析和生成(一般用于提升网络传输速率)。JSON 的官方 MIME 类型是 application/json,文件扩展名是.json。

MongoDB 使用 JSON（JavaScript ObjectNotation）文档存储记录。  
JSON 简单说就是 JavaScript 中的对象和数组，通过对象和数组可以表示各种复杂的结构。

**对象：**  
对象在 js 中表示为“{}”括起来的内容，数据结构为 {key： value,key： value,…}的键值对的结构，在面向对象的语言中， key 为对象的属性， value 为对应的属性值，所以很容易理解，取值方法为 对象.key 获取属性值，这个属性值的类型可以是 数字、字符串、数组、对象几种。
例如： {"FirstName":"ke","LastName":"me","email":"hikeme@aa"}
取值方式和所有语言中一样,使用 key 获取,字段值的类型可以是 数字、字符串、数组、对象几种。

### 2.1.2 BSON
BSON 是一种类 JSON 的一种二进制形式的存储格式，简称 Binary JSON，它和 JSON 一样，支持内嵌的文档对象和数组对象，但是 BSON 有 JSON 没有的一些数据类型，如 Date 和 BinData 类型。  

它的优点是灵活性高，但它的缺点是空间利用率不是很理想。  
BSON 有三个特点：轻量性、可遍历性、高效性。  
对JSON 来说，数据存储是无类型的，比如你要修改基本一个值，从 9 到 10，由于从一个字符变成了两个，所以可能其后面的所有内容都需要往后移一位才可以。而使用 BSON，你可以指定这个列为数字列，那么无论数字从 9 长到 10 还是 100，我们都只是在存储数字的那一位上进行修改，不会导致数据总长变大。当然，在 MongoDB 中，如果数字从整形增大到长整型，还是会导致数据总长变大的。

有时 BSON 相对 JSON 来说也并没有空间上的优势，比如对{“sex”:1}，在 JSON 的存储上,'1'只使用了一个字节，而如果用 BSON，那就是至少 4 个字节

## 2.2 MongoDB 特点
**高性能：** Mongodb 提供高性能的数据持久性，尤其是支持嵌入式数据模型减少数据库系统上的I/O 操作，索引支持能快的查询，并且可以包括来嵌入式文档和数组中的键

**丰富的语言查询：** Mongodb 支持丰富的查询语言来支持读写操作（CRUD）以及数据汇总，文本搜索和地理空间索引

**高可用性：** Mongodb 的复制工具，成为副本集，提供自动故障转移和数据冗余，水平可扩展性： Mongodb 提供了

**可扩展性**，作为其核心功能的一部分，分片是将数据分，在一组计算机上。

**支持多种存储引擎：** WiredTiger 存储引擎和、 MMAPv1 存储引擎和 InMemory 存储引擎

## 2.3 MongoDB 包含的程序
`MongoDB Drivers`  
官方 MongoDB 客户端库提供 C， C ++， C＃， Java， Node.JS， Perl， PHP，Python， Ruby和Scala 驱动程序的参考指南。

`MongoDB Stitch`   
为开发人员提供了一个 API 到 MongoDB 和其他后端服务。保持 MongoDB 的全部功能和灵性，同时受益于强大的系统来配置细粒度的数据访问控制。

`MongoDB Atlas`  
MongoDB 在云中部署，操作和扩展的最佳方式。适用于 AWS，Azure 和 Google Cloud Platform。轻松将数据迁移到 MongoDB Atlas，零停机

`MongoDB Cloud Manager`  
是一个用于管理 MongoDB 部署的软件包。 Ops Manager 提供 Ops Manager 监控和 Ops Manager 备份，可帮助用户优化群集并降低操作风险

`MongoDB Charts`  
可以最快速最简单的创建 Mongodb 可视化图表

`MongoDB Connector for BI`  
MongoDB 商业智能连接器（BI）允许用户使用 SQL 创建查询，并使用现有的关系商业智能工具（如 Tableau， MicroStrategy 和 Qlik）对其 MongoDB Enterprise 数据进行可视化，图形化和报告。

`MongoDB Compass`  
通过从集合中随机抽样一个文档子集，为用户提供其 MongoDB 模式的图形视图。采样文件可最大程度地降低对数据库的影响，并能快速产生结果。有关 抽样的更多信息

`MongoDB Spark Connector`  
使用连接器，您可以访问所有使用 MongoDB 数据集的 Spark 库：用 SQL 进行分析的数据集（受益于自动模式推理），流式传输，机器学习和图形 API。您也可以使用连接器与 Spark Shell。


## 应用场景
