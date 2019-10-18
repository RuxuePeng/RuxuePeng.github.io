---
layout: post
title:  "知识图谱风格的问答"
subtitle: Intro to Knowledge-based Question Answering  
date:   2019-10-27
category: QA
tags: 问答 Question-answering NLP 中文文章
image: >-
  /assets/img/for_posts/P14/thumbnail.png
optimized_image: >-
  /assets/img/for_posts/P14/thumbnail.png
author: ruxuepeng
paginate: true
---   
本文是聚焦在用事实就能回答的、答案简短的问题(**factoid question**).   

三种套路教电脑回答问题:
* [信息检索+阅读理解(Information Retrieval + Reading Comprehension)]
* 知识图谱(Knowledge-based)
* 混合工业风(Hybrid)

上次说了第一种, 今天说第二种和第三种.  

---
## 基于知识图谱的问答
将用户提出的问题看作是对知识库的查询(query).先将问题转化为查询语句,再使用知识库回答问题.  

## 结合信息检索和知识图谱的混合风问答  
工业界不会单纯使用一种方式,毕竟能抓到耗子的都是好猫.怎么work怎么来.  
比如IBM的DeepQA系统会给两种方式得到的回答打分,然后选择得分高的回答.  

##如何评价问答系统    


## Reference  
* [NLP Progress - Question answering](https://github.com/sebastianruder/NLP-progress/blob/master/english/question_answering.md)  
