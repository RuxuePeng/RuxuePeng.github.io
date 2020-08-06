Hi guys,  
COVID-19已经把我困在家里长达半年了,工作其实变得很忙,借着休年假的机会来跟大家重新“见面”.  
今天想简单聊聊`自动检测网络言语暴力`这个任务(`Hate Speech Detection`).  

## Intro  
为了保护大家共享的社交媒体上的交流空间，高效准确地检测出对他人侮辱、歧视、贬低的言论变得越来越重要.  

为什么不能简单地设置黑名单词语，然后过滤掉它们呢？  
因为要确定一句话是否带有歧视,需要了解对话双方、语境、上下文,有时候恶意的言论里并没有使用任何有明确诋毁意义的词语.   
```
e.g. 嘲笑一个人的声音  
Kermit the frog called and he wants his voice back
```

目前常见的思路是使用经典机器学习或者神经网络做有监督分类.  

## 数据集方面的挑战  
* 标注数据成本高  
原因是歧视言论在随机挑选的评论中往往占比很小,要生成大量positive label的数据很费时间  
* 不存在通用的数据集  
已有数据集限制在特定话题或针对特定人群  
为了减少时间，数据集往往是通过搜索争议度高的话题并收集评论得来的,这样的数据集不够随机和通用  
* 不存在一个成规模的数据集  
大家往往都是从Twitter, stackoverflow, reddit, Youtube等平台收集收据并手动标注. 这个就增加了benchmark不同模型的难度  

* 已有数据集主要是英语

## 预处理手段       
用情感分类器筛选出负面的句子  
用分类器排除不带主观色彩的句子

## 特征工程的常见思路    
###  初级
词语使用频率（忽略出现顺序）  
词语级别的N-gram  
字母级别的N-gram（效果比词语级别好）  
是否存在URL链接  
标点符号的频率  
词语的长度（英文）  
大写字母的频率  
自造词的使用频率（并不存在的词语）  
乱码的频率（non-alpha numeric characters）  

### 建模生成feature
* 聚类  
聚类之后的cluster id  

* LDA  
LDA(Latent Dirichlet Allocation)之后的topic id  

* embedding  
词语级别的embedding  
短语级别的embedding    
句子/段落级别的embedding(实验证明比较有效)  

* 情感分析  
SentiStrength等开源分类器计算出的 情绪两极化程度(polar intensity)  

* POS语法分析  
如果两个词之间距离比较远,可以通过POS类feature 保留它们之间的关系  
```
e.g. 犹太人是XX（某歧视类名词）  
可以得到如下feature:  
nsubj(某歧视名词, 犹太人)  
```
还可以计算某歧视词语与某人物同时出现在同一种POS关系里的频率    

### 词典类  
是否包含一些歧视类词汇  
这类feature单独使用效果比较差，一般作为baseline或者作为语境类feature的补充  
* 通用词典 https://www.noswearing.com/dictionary
* 种族类词典 https://en.wikipedia.org/wiki/List_of_ethnic_slurs
* LGBT类词典 https://en.wikipedia.org/wiki/List_of_LGBT_slang_terms
* 残疾类词典 https://en.wikipedia.org/wiki/List_of_disability-related_terms_with_negative_connotations  


### meta-data类  
数据来源的社交网络平台提供的信息有时候对预测帮助很大  
假设: 使用过语言暴力的账号可能会再犯  

用户是否曾经使用过语言暴力  
用户的信息历史中使用过的不当词语的总和  
用户性别（Dadvar et al., 2012; Waseem and Hovy, 2016 认为这特征可能有用）  

### 多模态类(multimodal)  
假设: 结合图片去看网络评论对分类有帮助  
pixel级别的图片feature  
从图片的文字解说(caption)中得到的feature  


## 常见分类器  
SVM, RNN(Mehdad and Tetreault (2016))  
有一步到位的分类器,也有结合一系列分类器形成pipeline的  





















Reference
---  
[One-step and Two-step Classification for Abusive Language Detection on Twitter](https://arxiv.org/pdf/1706.01206.pdf)
[Challenges and frontiers in abusive content detection](https://www.aclweb.org/anthology/W19-3509.pdf)  
[A Survey on Hate Speech Detection using Natural Language Processing](https://www.aclweb.org/anthology/W17-1101.pdf)
