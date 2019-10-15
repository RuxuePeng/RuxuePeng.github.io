---
layout: post
title:  "模板Post Template"
subtitle: Placeholder
date:   2019-10-14
category: QA
tags: 问答 Question-answering NLP 中文文章
image: >-
  /assets/img/for_posts/P14/thumbnail.png
optimized_image: >-
  /assets/img/for_posts/P14/thumbnail.png
author: ruxuepeng
paginate: true
---
目前对于问答的研究,主要在研究用事实或常识的就能回答的问题(factoid question).  
比如中国的货币是什么？人民币.    
比如Musée de l'Orangerie在法国的哪里？巴黎.  
比如哈利·波特的母亲叫什么名字？莉莉·伊万丝·波特.  
比如Jean Fragonard是什么风格的画家? 洛可可.    

本文也是聚焦在这种类型的问答上面.

三种套路教电脑回答问题:
* 信息检索+阅读理解(Information Retrieval + Reading Comprehension)
* 知识图谱(Knowledge-based)
* 混合工业风(Hybrid)

## 基于信息检索系统的问答  
(IR-based Question Answering)  
系统的目的是在网上或者已有的一堆文本里<ins>找到一小段话来回答用户的问题 </ins>.  

先从用户提出的问题出发(Question Processing), 首先使用信息检索系统找到相关的文本(Document and Passage Retrieval),然后使用深度学习的阅读理解模型从文本中截取(Answer Extraction)能回答问题的文段(span of text).  
<img src="/assets/img/for_posts/P14/IR_based_qa.png" alt="信息检索"/>

接下来将分为3个步骤逐个解说    
* 对提问的处理(Question Processing)  
* 文本检索(Document and Passage Retrieval)  
* 答案抽取(Answer Extraction)

### 对提问的处理(Question Processing)  
要将提问转化为信息检索系统能使用的查询条件,就必须从问题中抽取关键字.  
有些系统还会抽取答案的类型(answer type),比如应该回答人、地点、还是对时间的描述?  
替换位置(focus), 也就是提问中最有可能被答案替换的位置,比如哪里、什么.   
问题的类型(question type)比如提问属于数学题还是简答题.    
* 栗子  
Q: 中国省会人口最多  
Answer type: 城市  
Focus: 省会  

#### (1) Query Formulation  
对原始问题做处理,有时候会使得后面的信息检索步骤更容易.  
如果检索库是整个网络的话,不担心文本不够,可以直接使用原始问题.  
如果检索库的文本比较少,比如是公司内部的文本或者维基百科的文章,那么为了增加找到相关文本的概率,往往会通过重写问题(query reformulation)把问题变成陈述句,或者将提问展开成很多种不同的表达形式,希望其中一种能匹配到检索库中的相关文本.  
* query reformulation  
例子:where was iphone 11 released -> 重写成陈述句-> iphone 11 was released in  
注意这些重写虽然是自动的,但是规则都是手写的...




#### (2) Answer Type Detection  
好处是如果我们能知道答案的类型,就可以在检索时跳过没提及答案那一类实体的句子.  
比如提问<cite>哈利·波特的母亲叫什么名字</cite>的答案类型是人名.
* 标签设置  
答案的类型可以直接使用命名实体识别中的类别:PERSON, LOCATION, ORGANIZATION,...  
也可以使用带有树结构的**answer type taxonomy**  
以下是一个人工设计的answer type taxonomy, 当然它也可以自动生成  
<img src="/assets/img/for_posts/P14/answer_type_taxonomy.png" alt="信息检索"/>


* 模型  
现代一般使用有监督训练提问分类器,使用带有答案类型标签的数据集进行训练.  
可以是依赖特征的分类器,也可以是完全依靠神经网络的分类器.  

* 常用的特征  
提问句子中词语以及词语的词嵌入向量;  
每个词的part-of-speech标签;  
句子中的识别出的命名实体(named entities);  
某些关键提示词(answer type word/question headword)的出现与否,比如疑问词后面的词组.  
    * What is the state **flower** of California?
    * 哪个**城市**最适合秋天去？  





### 文本检索(Document and Passage Retrieval)

### 答案抽取(Answer Extraction)






## 基于知识图谱的问答
将用户提出的问题看作是对知识库的查询(query).先将问题转化为查询语句,再使用知识库回答问题.  

## 结合信息检索和知识图谱的混合风问答  
工业界不会单纯使用一种方式,毕竟能抓到耗子的都是好猫.怎么work怎么来.  
比如IBM的DeepQA系统会给两种方式得到的回答打分,然后选择得分高的回答.  

##如何评价问答系统    
