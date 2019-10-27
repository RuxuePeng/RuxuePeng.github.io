---
layout: post
title:  "用知识图谱做问答"
subtitle: Intro to Knowledge-based Question Answering  
date:   2019-10-27
category: QA
tags: 问答 Question-Answering NLP 中文文章
image: >-
  /assets/img/for_posts/P15/sky_above_lake.png
optimized_image: >-
  /assets/img/for_posts/P15/sky_above_lake.png
author: ruxuepeng
paginate: true
---  
[上一篇](https://oneepochaway.com/question-answering-intro/)介绍了如何利用Information Retrieval和阅读理解模型来做单步乃至多步推理问答系统.  

本文介绍另两种问答系统的思路:  
* [基于知识图谱的问答](#基于知识图谱的问答)  
* [结合TBQA和KBQA的工业QA](#结合TBQA和KBQA的工业QA)    

---
## 基于知识图谱的问答  
将提问文本转化为对数据库的查询语句，然后通过查询已有数据库得到问题的答案.  
其中语义解析器(semantic parser)负责将提问文本转化为按固定逻辑表达的查询语句, 而数据库可以是真正的关系型数据库，也可以是一堆三元组(RDF triples)[^1].  

以三元组为例,问答任务可以定义为根据提问找到对应的三元组，并返回提问所需的三元组成员.  
<img src="/assets/img/for_posts/P15/parser_example.png" alt="eg"/>  

### 基于规则的方案    
很常见的提问可以直接写规则找到三元组.  
比如问某个实体的生日日期,可以先正则表达式找到提问中的when和born,然后跑个命名实体识别得到实体名.  

### 有监督模型  
少数情况中我们有标注数据.  
* 对于每个训练集的提问, 都标注了它的逻辑表达式:  
    | Question  |  Logical form  |   
    |  加州的首府在哪里      | 首府在(加州, ?x) |  

模型会在训练中总结出提问的套路,形成对一类提问的解析规则(parsing rule).  
* e.g. 训练集中有很多对生日日期的提问:  
    When was Tom Hiddleston born -> birth-year(Tom Hiddleston, ?x)  
    When was Christopher Manning born -> birth-year(Christopher Manning, ?x)  
    ...  
那么训练后的模型会总结出如下解析规则  
<img src="/assets/img/for_posts/P15/supervised_example.png" alt="eg"/>  

也有论文证明过即使是较复杂的逻辑关系也可以这么学出来.  



### 半监督/无监督模型  
问题是大多数情况下我们都没有专门的标注数据.也没有数据库.  
所以模型一般都依赖于网络上的文本会有重复(textual redundancy).  

#### 步骤  
1. 先用**开放信息抽取(Open Information Extraction)** 来抽取网络文本中的三元组，形成一个庞大的知识库.  
英文抽取器有REVERB和它的升级版[OpenIE 5](https://github.com/dair-iitd/OpenIE-standalone).
    * "1732"这种对时间描述 可以用[SUTime](https://nlp.stanford.edu/software/sutime.shtml)标准化

2. 将提问文本转化为标准化的查询语句  
    * 实体转化为数据库中的某个概念  
    "Fragonard"用Entity Linking[^2]对应到维基百科的一页    
    * 实体间的词组转化为数据库中的某种关系  
    目前已经有人生成了一系列能对应到Freebase relation的[惯用词组列表](https://openie.allenai.org).  
    ```
    对应到country.capital关系的惯用词组:
    capital of, capital city of, become capital of, capitol of, national capital of, official capital of, ..., federal capital of, beautiful capital city of
    ```
    如果提问文本中的谓语词组能与列表中的词组对应，那么就对应上了Freebase的关系.  

    * 或者把提问按写好的模板转化为查询语句  
    | - Question Type - | --------- Pattern -------- | ---- Query ---- |  
    | Question (1-Arg.) | how big is **e** population | population(?, **e**)  |


3. 同义提问聚类  
利用[WikiAnswers语料库]训练同义句模型，将同义不同表达的提问对应到同一个查询语句上.  
可以参考[PARALEX模型](https://www.aclweb.org/anthology/P13-1158.pdf)
    * 像下面这堆提问应该被聚类到同一个群里, 然后通过查询找到同一个三元组`authored(milne,
winnie-the-pooh)`来回答  
<img src="/assets/img/for_posts/P15/paraphrase_cluster.png" alt="eg"/>  

---
## 结合TBQA和KBQA的工业QA












[^1]: 回忆到三元组是一种描述一对实体之间关系的形式, 比如  
    | Subject     |  Predicate  |  Object  |   
    |  Flora      | birth-month |    12    |  

[^2]: 回忆到Entity Linking任务，就是把一段字符对应到一个维基百科的页面.  

---
### Reference  
* [Speech and Language Processing - Question Answering](https://web.stanford.edu/~jurafsky/slp3/25.pdf)
* [Paraphrase-Driven Learning for Open Question Answering](https://www.aclweb.org/anthology/P13-1158.pdf)  
* [Open IE - AllenAI](https://openie.allenai.org/)
