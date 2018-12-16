---
title: lucene的使用
date: 2018-12-04 12:00:14
tags:
---

## 1，Lucene介绍

Lucene是一个开源的全文检索工具包，他的主要作用有两个：
1. 索引文档
2. 搜索文档

<!-- more -->

## 2，分词器（Analyzer）
> Analyzer是Lucene中比较重要的概念，在“索引文档”和“搜索文档”的时候都会用到Analyzer。

`Analyzer`类是一个抽象类，切分词的实现主要由子类实现，所以对不同的语言规则，要有不同的分词器。
`Analyzer`内部主要通过`TokenStream`类实现。
`Tokenizer`类和`TokenFilter`类是`TokenStream`的两个子类。
`Tokenizer`负责处理单个字符组成的字符流，读取`Reader`中的数据，处理后转成词汇单元。
`TokenFilter`完成文本过滤器的功能。

## 3，索引
> 索引过程就是把文档变成索引这种数据结构的过程，也可以说是创建倒排索引表的过程。

具体过程包括：创建Document、定义FieldType、设置Stored、设置Tokenized等。

## 4，查询
Lucene的查询方式有两种：
1. 使用查询表达式
> 可参考：http://www.cnblogs.com/xing901022/p/4974977.html

2. 使用Query API
> 具体可查询Lucene API或相关书籍

## 5，Lucene中一些深入的概念

### 1，倒排索引（Inverted Index）
倒排索引是一种索引方法，也是一种数据结构；被用来存储，在全文索引搜索下，某个单词在一个文档或一组文档中的存储位置的映射。

> 正排索引是以document id为key，其映射的内容记录着单词出现的次数、位置。
> 如果要搜索某个单词必须要扫描所有的document，才能正确的找到所有包含此单词的document，然后经过处理呈现给用户。
> 倒排索引是以每个单词为key，其映射的内容记录着所有包含此单词的document id的列表。
> 如果要搜索某个单词只需要扫描索引记录，就能直接找到所有正确的document呈现给用户。


### 2，向量空间模型（Vector Space Model）
*待完成*