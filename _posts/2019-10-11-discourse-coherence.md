---
layout: post
title:  "待定"
subtitle: English hook
date: 2019-10-11
categories: NLP
tags: NLP 中文文章
image:
optimized_image:
author: ruxuepeng
paginate: false
---

今天跟大家聊聊语篇连贯度分析~
## 本文要点
1. [简单介绍语篇连贯度分析](#简单介绍语篇连贯度分析)
2. [如何分析语篇的局部连贯度](#如何分析语篇的局部连贯度)
3. [如何分析不同文体的全局连贯度](#如何分析不同文体的全局连贯度)

---

## 简单介绍语篇连贯分析
Q1： 我们为什么在乎文本的连贯度高不高？  
A1： 任何需要保证文本输出质量的NLP产品都需要语篇连贯性分析. 比如信息抽取、自动摘要、机器翻译.

Q2: 什么是连贯(coherence)？  
A2: 连贯是一种句子之间的关系，定义了逻辑顺畅的语篇与随机排列的句子们之间的区别.  

Q3: 可以从哪些角度一个文本的连贯度？  
A3: 分为局部连贯(local coherence)以及全局连贯(global coherence).  
局部连贯是指语篇中连续N句话之间的关系，比如前后连接的句子对(span)之间的连贯关系;  
全局连贯是指站在段落乃至全文的高度，看语篇的行文结构对全文连贯度的影响.  

Q4: 什么是语篇连贯度分析任务？  
A4: 有很多方向，比如自动的对语篇进行解析，进而得到全文逻辑结构的图形表示；训练模型自动衡量文本的连贯度并打分.


---

## 如何分析语篇的局部连贯度
本部分包括：
1. [三种分析局部连贯的角度](#三种分析局部连贯的角度)
2. [如何解析句子间的连贯关系(coherence relation)](#如何解析语篇的连贯关系)
3. [如何追踪语篇当下的核心实体](#如何追踪语篇当下的核心实体)
4. [如何使用纯神经网络给语篇的连贯度打分](#如何使用纯神经网络给语篇的连贯度打分)

---
### 三种分析局部连贯的角度
一段话之所以连贯，可能是因为前后句子之间有逻辑关系，也可能是因为它们在讨论同一个现实世界的实体，还有可能是因为它们在讨论一个核心的话题.  
1. 句子之间存在连贯关系（Coherence Relations）  
句子之间可以存在各种各样的连贯逻辑关系.  
有RST树和PDTB两种不同风格的框架来句子之间不同的逻辑关系，第二部分会细讲.    
2. 存在Entity-based Coherence  
句子们在讨论同一个中心的实体（称之为Center）.  
这个研究角度会追踪一个语篇里面，目前被讨论的实体是什么，如果实体变来变去，显然这个语篇就不是一个很连贯的语篇.
3. 存在Topical Coherence  
句子们在讨论同一个话题，也就是使用同一个semantic field里面的词语
e.g. Before winter I built a **chimney**, and shingled the sides of my **house**... I have thus a tight shingled and plastered **house**... with a **garret** and a **closet**, a large **window** on each side.  

---
### 如何解析语篇的连贯关系
从第一个角度衡量一个文本的局部连贯度.
### (1) Theory
常用的理论框架是修辞结构理论(Rhetorical Structure Theory/RST）.  
RST分析的最小单位是一对句子或一对词组，句子/词组称为基础语篇单位(Elementary Discourse Unit/**EDU**)或者语篇段(discourse segment).

RST定义的每一类关系中，都定义了一个核心角色与一个卫星角色.  
**核心(nucleus)** 能够被独立理解，更接近作者的行文目的.  
**卫星(satellite)** 的内容一般需要结合核心去理解. (好生动有没有！)  

以下是一小部分RST定义的连贯关系：
* <em>起因Reason</em>  ——核心是一个行为动作，卫星是这个动作的起因  
	e.g. [Flora星期五没有来.]<sup>nucleus</sup> [因为她要去参加朋友的婚礼.]<sup>satellite</sup>
* <em>详述Elaboration</em>  ——核心是一个状态，卫星是对这个状态的补充描述  
	e.g. [Phoebe来自曼哈顿.]<sup>nucleus</sup>[她住在中央公园的西边.]<sup>satellite</sup>  
* <em>论据Evidence</em>  ——核心表达了一个观点，卫星是支撑的论据  
	e.g. [今天下班肯定会塞车.]<sup>nucleus</sup>[今晚有周杰伦演唱会]<sup>satellite</sup>  
* <em>语源Attribution</em> ——核心是一段话语，卫星指明了说话者是谁  
	e.g. [Frank跟我说]<sup>satellite</sup>[今天美术馆人会很少.]<sup>nucleus</sup>   
	e.g. [Analysts estimated]<sup>satellite</sup>[that sales at U.S. stores declined in the quarter]<sup>nucleus</sup>  
* <em>List</em> ——两个句子都是核心角色,关系是并列的  
	e.g.[他是班长.]<sup>nucleus</sup>[她是团支书.]<sup>nucleus</sup>  

RST树干:
<img src="/assets/img/for_posts/P12/RST_branch.png" alt="RST一根树干"/>

RST理论通过给一对对的EDU标注连贯关系，形成像这样解析全文的RST树：  
<img src="/assets/img/for_posts/P12/RST_tree.png" alt="RST树"/>  

要注意到这种语篇级别的RST树的标注是很昂贵的，像PDTB这样的数据集就不包含树，只包含句子级别的关系标注.  
所以也有只解析句子级别连贯度的解析器，这里简称为PDTB解析好了.

### (2) Dataset
* [Penn Discourse TreeBank/PDTB (最新2008)](https://catalog.ldc.upenn.edu/LDC2008T05)
包含18000个明确关系(explicit relation)，16000个隐含关系(implicit relation)
只包含每一对span的关系(span-pair relation)，不包含整个语篇级别的树.  
打标注的人被要求做选择题，在一堆关联词里面选一个作为输入句子对的连接词，选择有：
because, although, when, since, or as a result，然后这些关联词会与逻辑关系对应，比如because对应到因果关系.
* [RST Treebank (2002)](https://catalog.ldc.upenn.edu/LDC2002T07)  
从the Penn Treebank数据集里面选了385个英文文本（347训练/38测试)，以及它们的RST解析树
也就是会将标注整理为整个语篇层面的树状.  
* [Chinese Discourse TreeBank（2014)](https://catalog.ldc.upenn.edu/LDC2014T21) 中文的数据集.  
按照PDTB的风格给164个中文文档标注.
由于中文里面明确关联词的使用率比英语低，这个数据集直接给每一对语篇段打关系标注，有11种不同的标签.  
* [MCDTB A Macro-Level Chinese Discourse TreeBank (2018)](https://www.aclweb.org/anthology/C18-1296.pdf)
中文的数据集.
考虑段落之间的关系, 有像RST树的语篇级别的标注.
<img src="/assets/img/for_posts/P12/MCDTB.png" alt="MCDTB数据集"/>

### (3) 如何解析出RST树  
为了要把语篇解析成RST树，需要两步：  
第一步：**基础语篇单位切分（EDU segmentation）**  
——把文本切成小块小块，或者叫语篇切分(discourse segmentation)

第二步： **对EDU们做RST解析 (RST Parsing)**  
——从下往上合并EDU，自动生成一棵树

a. 如何把文本切成小块?  —— 有监督学习序列标注模型  
使用RST Discourse Treebank数据集，训练一个序列标注(sequential labelling)模型.被标注为1的位置表示从这里切断句子，分成前后两个EDU.  
模型的设计就是典型序列标注任务的套路了: BiLSTM-CRF.  
今天以一篇[EMNLP 2018的论文](https://www.aclweb.org/anthology/D18-1116.pdf
)为例，看看常用的模型结构.
<img src="/assets/img/for_posts/P12/EDU_segmentation_model.png" alt="序列标注模型做文本切分"/>
从下往上看：  
a. 拿到词嵌入向量  
先把词过ELMO，得到向量表示*r*<sub>1</sub>，再跟另外一个向量表示*e*<sub>1</sub>粘起来，得到输入（黄色与绿色）
* why?
由于RST的数据集很小，从零开始训练显然效果会差，于是用了预训练的ELMO.

b. 得到词语上下文信息  
过个BiLSTM，然后把每个time step得到的两个hidden state向量粘起来

c. 允许模型重视上下文中的特定位置  
在一个限定长度的窗口里面算注意力*a*<sub>n</sub>，也黏到bi-LSTM的输出上，输出*h*<sub>n</sub>
<p><span class="math inline">\(\tilde{r_{1}}\)</span></p>


c. 判断每个位置应不应该切开  
最后每个位置的标签由CRF模型决定，求的是序列走到这一步时所有可能标签的条件概率


---
### 如何追踪语篇当下的核心实体

---
### 如何使用纯神经网络给语篇的连贯度打分

---

### 小结
placeholder

---
## 如何分析不同文体的全局连贯度

### Reference
* Paper name - arxiv url
