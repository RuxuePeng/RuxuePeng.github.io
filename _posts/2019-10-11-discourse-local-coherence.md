---
layout: post
title:  "句子之间的连贯程度如何衡量？"
subtitle: Ways to analyze the local coherence of text
date: 2019-10-12
categories: NLP
tags: 语篇连贯 Discourse-coherence NLP 中文文章
image:
optimized_image:
author: ruxuepeng
paginate: false
---

[上篇](https://oneepochaway.com/discourse-coherence-intro/)讨论了全局连贯性.今天跟大家聊聊语篇局部连贯度分析~  
局部连贯是指语篇的句子级别的连贯度,比如前后连接的句子对之间是否连贯.有三种分析局部连贯的角度, 我们将介绍每一个研究角度下的成果.      

## 本文要点
1. [如何解析句子间的连贯关系(coherence relation)](#如何解析句子间的连贯关系)
2. [如何追踪语篇当下的核心实体](#如何追踪语篇当下的核心实体)
3. [如何使用纯神经网络给语篇的连贯度打分](#如何使用纯神经网络给语篇的连贯度打分)


---
## 如何解析句子间的连贯关系
这个角度的出发点是如果两个句子之间存在某种逻辑关系,那么这段话是连贯的.  
* [理论框架](#理论框架)  
* [数据集](#数据集)  
* [解析树](#解析树)
* [只解析句子对](#只解析句子对)  

### 理论框架
常用的理论框架是修辞结构理论(Rhetorical Structure Theory).接下来简称**RST**.    
RST分析的最小单位是一对句子或一对词组,句子/词组称为基础语篇单位(Elementary Discourse Unit/**EDU**)或者语篇段(discourse segment).

RST定义的每一类关系中,都定义了一个核心角色与一个卫星角色.  
**核心(nucleus)** 能够被独立理解,更接近作者的行文目的.  
**卫星(satellite)** 的内容一般需要结合核心去理解. (好生动有没有！)  

以下是一小部分RST定义的连贯关系：
* <em>起因Reason</em>  ——核心是一个行为动作,卫星是这个动作的起因  
	e.g. [Flora星期五没有来.]<sup>nucleus</sup> [因为她要去参加朋友的婚礼.]<sup>satellite</sup>
* <em>详述Elaboration</em>  ——核心是一个状态,卫星是对这个状态的补充描述  
	e.g. [Phoebe来自曼哈顿.]<sup>nucleus</sup>[她住在中央公园的西边.]<sup>satellite</sup>  
* <em>论据Evidence</em>  ——核心表达了一个观点,卫星是支撑的论据  
	e.g. [今天下班肯定会塞车.]<sup>nucleus</sup>[今晚有周杰伦演唱会]<sup>satellite</sup>  
* <em>语源Attribution</em> ——核心是一段话语,卫星指明了说话者是谁  
	e.g. [Frank跟我说]<sup>satellite</sup>[今天美术馆人会很少.]<sup>nucleus</sup>   
	e.g. [Analysts estimated]<sup>satellite</sup>[that sales at U.S. stores declined in the quarter]<sup>nucleus</sup>  
* <em>List</em> ——两个句子都是核心角色,关系是并列的  
	e.g.[他是班长.]<sup>nucleus</sup>[她是团支书.]<sup>nucleus</sup>  

RST树干:
<img src="/assets/img/for_posts/P12/RST_branch.png" alt="RST一根树干"/>

RST理论通过给一对对的EDU标注连贯关系,形成像这样解析全文的RST树：  
<img src="/assets/img/for_posts/P12/RST_tree.png" alt="RST树"/>  

要注意到这种语篇级别的RST树的标注是很昂贵的,像PDTB这样的数据集就不包含树,只包含句子级别的关系标注.  
所以也有只解析句子级别连贯度的解析器,这里简称为PDTB解析好了.

### 数据集
* [Penn Discourse TreeBank/PDTB (最新2008)](https://catalog.ldc.upenn.edu/LDC2008T05)
包含18000个明确关系(explicit relation),16000个隐含关系(implicit relation)
只包含每一对span的关系(span-pair relation),不包含整个语篇级别的树.  
打标注的人被要求做选择题,在一堆关联词里面选一个作为输入句子对的连接词,选择有：
because, although, when, since, or as a result,然后这些关联词会与逻辑关系对应,比如because对应到因果关系.
* [RST Treebank (2002)](https://catalog.ldc.upenn.edu/LDC2002T07)  
从the Penn Treebank数据集里面选了385个英文文本(347训练/38测试),以及它们的RST解析树
也就是<ins>会将标注整理为整个语篇层面的树状</ins>.  
* [Chinese Discourse TreeBank(2014)](https://catalog.ldc.upenn.edu/LDC2014T21) 中文的数据集.  
按照PDTB的风格给164个中文文档标注.
由于中文里面明确关联词的使用率比英语低,这个数据集直接给每一对语篇段打关系标注,有11种不同的标签.  
* [MCDTB A Macro-Level Chinese Discourse TreeBank (2018)](https://www.aclweb.org/anthology/C18-1296.pdf)
中文的数据集.
考虑段落之间的关系, 有像RST树的语篇级别的标注.
<img src="/assets/img/for_posts/P12/MCDTB.png" alt="MCDTB数据集"/>

### 解析树
如何语篇解析成RST树?  
第一步：[基础语篇单位切分(EDU segmentation)](#如何把文本切成小块)  
——把文本切成小块小块,或者叫语篇切分(discourse segmentation)  
第二步： [对EDU们做RST解析 (RST Parsing)](#如何做修辞结构解析)  
——从下往上合并EDU,自动生成一棵树

---
#### 如何把文本切成小块  
使用RST Discourse Treebank数据集,训练一个序列标注(sequential labelling)模型.被标注为1的位置表示从这里切断句子,分成前后两个EDU.  
模型的设计就是典型序列标注任务的套路了: BiLSTM-CRF.  

今天以一篇[EMNLP 2018的论文](https://www.aclweb.org/anthology/D18-1116.pdf
)为例,看看常用的模型结构.
<img src="/assets/img/for_posts/P12/EDU_segmentation_model.png" alt="序列标注模型做文本切分"/>
从下往上看：  
a1. 拿到词嵌入向量  
先把词过ELMO,得到向量表示*r*<sub>1</sub>,再跟另外一个向量表示*e*<sub>1</sub>粘起来,得到输入(黄色与绿色)
* why?
由于RST的数据集很小,从零开始训练显然效果会差,于是用了预训练的ELMO.

a2. 得到词语上下文信息  
过个BiLSTM,然后把每个time step得到的两个hidden state向量粘起来

a3. 允许模型重视上下文中的特定位置  
在一个限定长度的窗口里面算注意力*a*<sub>n</sub>,也黏到bi-LSTM的输出上,输出
<p><span class="math inline">\(\tilde{h_{n}}\)</span></p>


a4. 判断  
最后每个位置的标签由CRF模型决定,求的是序列走到这一步时所有可能标签的条件概率

看到这里你肯定猜到了,没错又有人把词嵌入从ELMO换成了BERT然后刷了一下准确率...
<img src="/assets/img/for_posts/P12/BERT_on_EDU_segmentation.png" alt="BERT做词嵌入"/>
[论文传送门](https://www.aclweb.org/anthology/W19-2715.pdf)  
最右边一列就是BERT,BERT在中文数据集上表现平平,在其他语言上BERT的表现很好.  
论文原话：<cite>BERT contextual embeddings beats all other systems on all datasets – except the Chinese RST treebank –, often by a large margin.</cite>

---
#### 如何做修辞结构解析  

#### 理论框架
从1999年以来, RST解析都是用**shift-reduce parsing**模型框架  

Shift-reduce Parser的基本设定  
* 需要一个队列Queue与一个栈Stack  
* shift动作: 把队列最前面的EDU的放进栈里面  
* reduce(l, d)动作: 把栈最上面的两个成员merge成一个成员
    * l 指的是连贯关系的标签,d指的是卫星-核心的方向(nuclearity direction), d ∈ {NN,NS,SN}
    * NN = 两个EDU都是核心(关系箭头是双向的/无向的)
    * NS = 左边的EDU是核心 右边的EDU是卫星(关系箭头指向左边的EDU)
    * SN = 左边的EDU是卫星 右边的EDU是核心(关系箭头指向右边的EDU)  
* 栗子  
假设我们现在目标是把如下的RST树建起来：
<img src="/assets/img/for_posts/P12/sample_RST_tree.png" alt="RST树"/>
[图片来源](https://www.aclweb.org/anthology/C18-1047.pdf)   
可以看到一共有4句话:  
*e*<sub>1</sub>, *e*<sub>2</sub>, *e*<sub>3</sub>, *e*<sub>4</sub>,  
两种连贯关系：attribution和elaboration  
需要对队列里面的EDU做如下操作, bottom-up地把树建出来:  
<img src="/assets/img/for_posts/P12/RST_parse.png" alt="RST解析"/>
[图片来源](https://www.aclweb.org/anthology/C18-1047.pdf)  
解析器每一步能做的就是从两个Action中选一个做：SH - shift或者RD - reduce  


#### 基本框架知道了,那么模型怎么知道选哪个action?  
之前的parser是靠一些设定的规则选择shift和reduce.  
近几年出现了神经网络风格的RST解析器,今天以[论文Transition-based Neural RST Parsing with Implicit Syntax Features](https://www.aclweb.org/anthology/C18-1047.pdf)为例, 看如何先得到句子向量表示,然后有监督地训练序列分类模型选择action.  
* 如何得到句嵌入向量——建一个编码器(encoder)
	* 假设你现在有一段文本,有词语 *w*<sub>1</sub>, *w*<sub>2</sub>, …, *w*<sub>t</sub>  
	* 它先用常规操作得到每个词的向量表示(比如字母级别嵌入模型、词嵌入模型、好几个模型粘起来,etc);  
	* 过Bi-LSTM得到 *h*<sub>1</sub>, *h*<sub>2</sub>, …, *h*<sub>t</sub> ;  
	* 过average pooling得到这段文本不同长度的表示*x*<sub>1</sub><sup>e</sup>, *x*<sub>2</sub><sup>e</sup>, …, *x*<sub>n</sub><sup>e</sup> ;  
	* <p><span class="math inline">\(x^{e} = \frac{1}{t-s+1}\sum_{k=s}^{t} h_{k}^{w}\)</span></p>  
	* 然后这些不同长度的表示*x*<sub>1</sub><sup>e</sup>, *x*<sub>2</sub><sup>e</sup>, …, *x*<sub>n</sub><sup>e</sup> 过第二层Bi-LSTM得到最终的这段文本的表示 *h*<sub>1</sub><sup>e</sup>, *h*<sub>2</sub><sup>e</sup>, …, *h*<sub>n</sub><sup>e</sup> .  

* 如何选择action——前馈神经网络  
	* 分类器是一个简单的前馈神经网络.网络的输入是栈最上面的三个成员 *s*<sub>0</sub>,*s*<sub>1</sub>,*s*<sub>2</sub>的表示向量 以及 队列下一个的成员的表示向量
	* <p><span class="math inline">\(o = W(h_{s0}^{t},h_{s1}^{t},h_{s2}^{t},h_{q0}^{e} )\)</span></p>  
	* 注意栈里面的成员有可能是带树叶的树干,所以每个成员的表示向量是成员那根树枝里面所有EDU的表示向量average pooling之后的结果,队列*q*<sub>0</sub>里面的成员因为一定是单个EDU,所以直接用上面encoder的输出就好.

* 如何训练——需要把树扁平成操作
    * 首先要把正确答案的RST树转化成一系列的shift,reduce操作.
    * 然后既然是分类器,就把输出的action数值softmax一下转化成操作的概率分布*p*<sub>action</sub>,使用交叉熵作为损失函数.

* 如何评价——RST-Pareval metrics
    * 一般会把正确答案的RST树转化成向右展开的二叉树,然后算四样东西：
    trees with no labels (S for Span), labeled with nuclei (N), with relations (R), or both (F for Full), 对这四个评价指标都算micro-averaged F1.  

---
## 只解析句子对
通常被称为浅层语篇解析(shallow discourse parsing),因为任务只涉及span对子之间的联系.而不是像RST解析一样,目标是得到语篇的一整颗树.  
使用PDTB这种不包含树结构标注的数据集.  

#### 四种不同的子任务
1. 区分有连贯作用的关联词与没有连贯作用的词  
	e.g1. Selling picked up as previous buyers bailed out of their positions **and** aggressive short sellers—anticipating further declines—moved in.  
	and这个关联词有连贯作用, 它用elaboration的关系连接了两个句子  
    e.g2. My favorite colors are blue **and** green  
    and这个词在这只是两个词的连接, 没有连贯作用    
    以前这个任务是语言学的路子做,今年有了序列分类的做法: 标签是IOB类标签,用CRF+Bi-LSTM    
2. 找到关联词连接的两个句子(span)  
还是序列标注模型    
3. 给任务2中找到的关联词联系起来的两个句子,打关系标签(Coherence Relation Assignment)  
很难的一个任务,已有的方法是[使用提示词/提示短语(cue phrase)](http://demo.clab.cs.cmu.edu/NLP/S19/files/slides/23-discourse_pragmatics_entity_linking.pdf):  
先将句子中的提示词找到; 将文本做语篇切分,得到一个个EDU; 对于相邻的两个EDU,对它俩的关系进行分类  
4. 给任意一对前后句子打关系标签(Implicit Relation Prediction)  
这个子领域目前的[SOTA](https://arxiv.org/pdf/1710.04334.pdf)怎么做的呢？简单而有效的一个模型:  
用BERT得到两个span的表示,取<CLS> token对应的timestep的hidden state, 过一层tanh,再过一层softmax,得到sense分类的概率分布.  

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
