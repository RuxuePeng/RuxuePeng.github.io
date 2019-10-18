---
layout: post
title:  "9012年了，你还没学过问答和阅读理解?"
subtitle: Intro to Reading Comprehension and Information-Retrieval-based Question Answering  
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
目前对于问答的研究,主要在研究用事实就能回答的、答案简短的问题(**factoid question**).  
比如中国的货币是什么？人民币.    
比如Musée de l'Orangerie在法国的哪里？巴黎.  
比如哈利·波特的母亲叫什么名字？莉莉·伊万丝·波特.  
比如Jean Fragonard是什么风格的画家? 洛可可.    

本文也是聚焦在这种类型的问答上面.

三种套路教电脑回答问题:
* [信息检索+阅读理解(Information Retrieval + Reading Comprehension)](#基于信息检索的问答系统)
* 知识图谱(Knowledge-based)
* 混合工业风(Hybrid)

本文由于篇幅有限, 先看第一种: 基于信息检索的问答.  

---
## 基于信息检索的问答系统
(IR-based Question Answering)  
目的是在网上或者已有的一堆文本里<ins>找到一小段话(span)来回答用户的问题 </ins>.  
基本框架如下:  
先从用户提出的问题出发(Question Processing), 首先使用信息检索系统找到相关的文本(Document and Passage Retrieval),然后使用深度学习的阅读理解模型从文本中截取(Answer Extraction)能回答问题的文段(span of text).  
<img src="/assets/img/for_posts/P14/IR_based_qa.png" alt="信息检索"/>

接下来将分为3个步骤逐个解说    
* [对提问的处理(Question Processing)](#对提问的处理)  
* [文本检索(Document and Passage Retrieval)](#文本检索)  
* [答案抽取(Answer Extraction)](#答案抽取)  

最后将聚焦到**如何使用深度学习模型做答案抽取**,aka[如何做阅读理解任务](#阅读理解).

---
## 对提问的处理
(Question Processing)  
要将提问转化为信息检索系统能使用的查询条件,就必须从问题中抽取关键字.  
有些系统还会抽取答案的类型(answer type),比如应该回答人、地点、还是对时间的描述?  
抽取替换位置(focus), 也就是提问中最有可能被答案替换的位置,比如“哪里”、“什么”.    

### (1) Query Formulation  
对原始问题做处理,有时候会使得后面的信息检索步骤更容易.  
如果检索库是整个网络的话,不担心文本不够,可以直接使用原始问题.  
如果检索库的文本比较少,比如是公司内部的文本或者维基百科的文章,那么为了增加找到相关文本的概率,往往会通过重写问题(query reformulation)把问题变成陈述句,或者将提问展开成很多种不同的表达形式,希望其中一种能匹配到检索库中的相关文本.  
* query reformulation  
例子:where was iphone 11 released -> 重写成陈述句-> iphone 11 was released in  
注意这些重写虽然是自动的,但是规则都是手写的...




### (2) Answer Type Detection  
好处是如果我们能知道答案的类型,就可以在检索时跳过没提及答案那一类实体的句子.  
比如提问<cite>哈利·波特的母亲叫什么名字</cite>的答案类型是人名.  

**标签设置**  
答案的类型可以直接使用命名实体识别中的类别:PERSON, LOCATION, ORGANIZATION,...  
也可以使用带有树结构的**answer type taxonomy**  
以下是一个人工设计的answer type taxonomy, 当然它也可以自动生成  
<img src="/assets/img/for_posts/P14/answer_type_taxonomy.png" alt="信息检索"/>


**模型**  
现代一般使用有监督训练提问分类器,使用带有答案类型标签的数据集进行训练.  
可以是依赖特征的分类器,也可以是完全依靠神经网络的分类器.  
github项目: [Keras示例](https://github.com/tim5go/cnn-question-classification-keras/blob/master/main.py), [Torch示例](https://github.com/kearnsw/question-type-classification/blob/master/code/sc/models/sentence_classifier.py)

**数据集**  
* [BQuLD](https://github.com/tim5go/cnn-question-classification-keras/blob/master/data/question_labels.json)  
1216个问题，及其答案. 问题的繁体中文与英文翻译都有. 包含了答案的类别.  
* [TREC](https://trec.nist.gov/data/qa.html)  
包含了1999年-2004年的6个数据集, 包含了答案的类别.  
* [Upenn 实验数据](https://cogcomp.seas.upenn.edu/Data/QA/QC/)  
1000-5500个英文问题, 包含了答案的类别.  
* [Yahoo lab数据集](https://webscope.sandbox.yahoo.com/catalog.php?datatype=l)
    * L5 - Yahoo! Answers Manner Questions
    142627个问题及其答案，还有问题的分类  
    * L6 - Yahoo! Answers Comprehensive Questions and Answers
    4483032个问题及其答案，还有问题的分类  

**常用的特征**  
* 提问句子中词语以及词语的词嵌入向量;  
* 每个词的part-of-speech标签;  
* 句子中的识别出的命名实体(named entities);  
* 某些关键提示词(answer type word/question headword)的出现与否,比如疑问词后面的词组.  
    * What is the state **flower** of California?
    * 哪个**城市**最适合秋天去？  

---
## 文本检索
(Document and Passage Retrieval)  
从全部文本里选出备选文本和文本中的备选语段.    
<img src="/assets/img/for_posts/P14/passage_retrieval.png" alt="文本检索"/>  
将提问中抽取出的信息输入信息检索系统.  
同时把所有备选的文本输入系统，让系统按照相关性对文本排序. 将最相关的N个文本取出. 至此Document Retrieval就做完了.  
然后将这N个文本切分成段落、句子、甚至语段, 进一步筛选可能的答案,
最后输出备选语段,Passage Retrieval就做完了.  

### 如何筛选语段  
* 对语段做命名实体识别  
得到每个语段包含的实体以及类别, 结合提问中利用Answer Type Classification抽取的答案类别，对语段进行筛选.  
* 训练网络对语段与提问的相关度进一步打分  
特征可以包括: 比如提问的答案应该是一个地名，将语段中地名类实体的个数、  
提示关键词的个数、所在文本的排名、与提问重合的N-gram个数等等.

来看2018年的[AllenAI的BiDAF模型](https://arxiv.org/pdf/1611.01603.pdf)如何从文本中选语段(span)  
<img src="/assets/img/for_posts/P14/BIDAF_model.png" alt="文本检索"/>  
由于我找到了写得很清晰的关于这个模型的[中文笔记](http://www.shuang0420.com/2018/04/01/%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0%20-%20Bi-Directional%20Attention%20Flow%20for%20Machine%20Comprehension/), 这边就不再展开说了.  
它使用了改良的注意力机制,预测文本中的每个位置作为回答的开始位置与结束位置的概率.属于有监督学习.  
损失函数利用了模型对正确答案的开始位置*y*<sub>i</sub><sup>1</sup>与结束位置*y*<sub>i</sub><sup>2</sup>的概率预测:  
 <p><span class="math inline">
 \(L(\theta) = -\frac{1}{N}\sum_{i}^{N}[ log(p_{y_{i}^{1}}^{1} + log(p_{y_{i}^{2}}^{2})]\)</span></p>

[官方代码-Tensorflow 1.2版本](https://github.com/allenai/bi-att-flow/tree/dev)

---
## 答案抽取
(Answer Extraction)  
任务是从一句话或者一个段落里面找到最能回答提问的语段.    
常转化为语段分类任务做: 给一个语段, 决定它里面是否包含提问的答案.  

e.g. 哪家公司是湾区出了名的血汗工厂？  
<ins>亚马逊</ins>尝试用各种实验性管理方式逼出员工的潜力，但部分现任和离职员工抱怨公司一些......

### 旧模型们  
1. 基于规则(Pattern Matching Parser)   
根据每种不同的答案类型，设计常见的pattern，然后抽取固定位置的语段.  
e.g. 答案是生日日期  
Pattern: [QP] was born on [AP]  
这个模型的一个明显问题是模型可复制性很差，换了个答案类型又要重新设计pattern了  
2. 使用训练好的命名实体识别模型(Named Entity Tagger)  
e.g. 你知道提问的答案会是一个地点，就用命名实体识别模型把文本中包含地点实体的语段都找出来.  
这个模型的一个明显问题是不能解决答案类型与命名实体类别不同的提问，比如定义类提问.   
3. 基于特征的分类(Feature-based Classifier)    
从备选语段中得到以下特征： 是否出现了属于答案类别的实体; 与提问关键字的重合词数；是否包含在提问中没有的新词；语段是否紧跟逗号句号问号感叹号；语段是否是提问中的词的同位语等.  
4. 词组拼贴 (N-gram tiling)  
从文本中生成unigram, bigram, trigram. 给N-gram们按照出现频率,有多接近答案的类型打分.  
把用词重复N-gram融合为一个更长的语段,分数叠加.  
不断地整合,同时把低分的语段去掉，直到没有可以整合的语段了，就输出最高分的语段作为答案.   
贪心的算法会先从高分的语段开始tile.
<img src="/assets/img/for_posts/P14/N-gram_tiling.png" alt="答案抽取tiling"/>
在网络搜索中使用,因为网上重复的文本很多,可以融合很多次.   

现代方法使用神经网络从文本中抽取能作为提问的答案的语段, 常被转化为**阅读理解任务**.  

---
## 阅读理解
 模型不仅能用来测量NLU的表现，还可以作为问答系统中的**阅读器(reader)** .  
 出发点: 提问和答案是语义上相似的两个文本.

### 数据集
[SQuAD 2.0](https://rajpurkar.github.io/SQuAD-explorer/)  
基于英文维基百科文章内容设计提问,并文中人工抽取语段作为回答,形成的数据集.  
2018年的2.0版本中加入了无法被回答的问题这一个新的类别. 一共15万个问题.（看到数据集也这么积极地迭代版本真是<del>羡慕斯坦福有钱</del>佩服)  

左边是关于亚马逊雨林的文章，右边是提问以及黄金答案.
<img src="/assets/img/for_posts/P14/squad.png" alt="squad"/>  

有些问题通过阅读这个文章并不能回答  
比如上面的例子  
问: 雨林在哪个时期没能繁茂生长?  
(The rain force failed to thrive during what periods?)  
此时黄金答案为无答案(No Answer)  
通过文章可知雨林在困难时期也在繁茂生长，这个问题的出发点就是错的，因此属于无法回答的问题.  

[NewsQA](https://www.microsoft.com/en-us/research/project/newsqa-dataset/#!download)  
基于[DeepMind提供的CNN Dailymail数据集](https://cs.nyu.edu/~kcho/DMQA/)中的新闻文章整理出的10万个问题. 跟SQUAD一样，问题和答案都是人设计人抽取的.包含不可回答的问题.  

[WikiQA](https://www.microsoft.com/en-us/download/details.aspx?id=52419)  
3047个问题.不包含语段答案，只包含完整句子作为答案.  

## 如何评价问答系统    


## Reference  
* [Know What You Don’t Know: Unanswerable Questions for SQuAD](https://arxiv.org/pdf/1806.03822.pdf)  
* [NewsQA: A Machine Comprehension Dataset](https://arxiv.org/pdf/1611.09830.pdf)  
* [WikiQA: A Challenge Dataset for Open-Domain Question Answering](https://www.aclweb.org/anthology/D15-1237.pdf)  
* [University of Washington - LING 573 slides](http://courses.washington.edu/ling573/SPR2014/slides/ling573_class13_ae_flat.pdf)  
* [NLP Progress - Question answering](https://github.com/sebastianruder/NLP-progress/blob/master/english/question_answering.md)  
