---
layout: post
title:  "强化学习101"
subtitle: Reinforcement Learning 101  
date:   2020-01-05 10:42:00
category: RL
tags: Reinforcement-Learning 强化学习 中文文章 Deep-Learning
image: >-
  /assets/img/for_posts/P16/ChristmasBreak2019.JPG
optimized_image: >-
  /assets/img/for_posts/P16/ChristmasBreak2019.JPG

author: ruxuepeng
paginate: true
---

最近换了游戏公司的工作,看同事们隔三差五讨论RL,于是Flora偷偷来补习啦.  

## 本文要点

* 强化学习系统101    
* 强化学习系统的几种分类角度    

---

## 基本概念 - Concepts
* Agent - 智能体
* Interaction - （智能体与环境之间的）互动  
* Environment - 环境
* State - （环境的）状态
* Action - (某状态下可以采取的)行动
* Value - （某行动的）价值,一般是（基于某行动）能得到奖励的期望
* Value Function - 预测每个行动的价值的函数
* Policy - （某状态下）应该选择哪种行动的行为策略
* Policy Function - 给出每种备选行动被选中的概率的函数  
* Self-play  不存在真实对手,两个相同的模型对打  
* Greedy Move 单步选择时选价值最高的行动(exploitation)    
* Exploration Move  不选价值最高的行动,为别的行动贡献经验数据   


---

## 强化学习101  
强化学习的本质就是学习如何作反应.强化学习系统通过与环境的互动,学会根据周围环境的状态作出合适的反应.这是它最接近人类学习方式的一点.  

### 与机器学习其他方向的区别     
与有监督学习(Supervised Learning)最大的不同是,强化学习的系统没有一个明确的“老师”(Supervisor).它是通过不断尝试不断翻车的过程进步,也就是**Trial-and-error Search** .  
跟人一样,你现在做的选择可能不会对你的生活有立刻的影响,但是在未来的某一天,你发现这个决定其实改变了你人生的方向.   
这就是RL的第二个特点,奖励可能延迟发生(**Delayed Reward**) .  

### 智能体(Agent）   
是强化学习系统的一个部分.一般满足这些要求:  
* 能感受到环境目前的状态(State)
* 能作出影响环境状态的行动(Action)  
* 有一个目标（Goal）,而且这个目标跟它周围的环境状态相关  

比如,忘记给你过生日男朋友是智能体吗?  
不是,因为他不能感受到环境目前的状态.  

又比如,记得你生日但是没有给你过生日的男朋友是智能体吗?  
不是,因为他不能做出行动,或者他的目标跟环境这个状态不相关.  

咳咳...

---
## 强化学习系统的几种分类角度  
### 1. 下一步的选择如何决定 —— Policy vs Value  
强化学习系统可以分为基于策略的(Policy-based)和基于价值的(Value-based).  

比如说,Flora去超市买东西,结账可以选择去人工柜台或者自助柜台.  
Policy-based的设定可能是  
去人工柜台的概率 40%  
去自助柜台的概率 60%

而Value-based的设定可能是  
去人工柜台的价值是 1  
去自助柜台的价值是 2  

然后通过学习,在不同状态下这里的数字会被更新.   
如果环境需要智能体采取连续好几个行动,基于策略的系统就能把行动的概率累乘计算,而基于价值的系统就不能自然地计算价值了.  

Policy Gradients算法属于基于策略的.  
Q-learning算法、Sarsa算法都是基于价值的.  
**Actor-Critic算法**, Actor按概率选择行动,是基于策略的; Critic对行动进行打分,是基于价值的,所以属于混合算法.  

### 2. 系统更新方式 —— 回合 vs 单步  
Monte-Carlo **回合更新** <del>aka从谈恋爱开始到分手,最后总结所有历史行为,只更新一次策略</del>  
Temporal Difference **单步更新** <del>aka一边谈恋爱一边总结经验</del>.  

明显逻辑上来说,单步更新更好 <del>分了手才总结还有什么用</del>  

### 3. 经验数据的来源
* On-Policy  
一边打游戏一边学习如何更好地打游戏  
* Off-Policy  
看别人打游戏学习如何更好地打游戏  
先打一段时间游戏, 再观看自己打游戏的资料学习  
总之离线训练时,经验数据的来源没有什么限制.  

举个例子, Q-Learning是一种Off-Policy算法,因为会看哪个动作在本状态下能取得最大价值maxQ,但是更新了Q表之后, 并不会直接走这一步.  
Sarsa跟Sarsa(lambda)算法就属于On-Policy, 因为会直接走这一步,然后用动作在本状态下得到的真实的奖励更新Q表.也就是大家说的“说到做到”.   
Deep Q Network也是一种离线算法.   

### 4. 是否对环境建模  
* Model-free System  直接与真实环境交互的系统
    - 比如Q-learning, Sarsa, Policy Gradients
* Model-based System 尝试对环境建模的系统
    - 系统利用模拟环境,尝试预判出所有行动的结果,并选择最优  
    - 比如送花还是送巧克力,系统预判送花的话、花粉过敏的女朋友会生病,预判送巧克力会加分,就会选择送巧克力
    - 而Model-free系统由于没有预判能力,一定概率会送花 <del>然后被打</del>


## 总结
介绍了强化学习最基础的一些概念跟分类,希望有帮到你ヾ(￣▽￣)

---
## Reference
* [Reinforcement Learning: An Introduction - 2nd Edition](https://web.stanford.edu/class/psych209/Readings/SuttonBartoIPRLBook2ndEd.pdf)
* [Soft Actor-Critic](https://arxiv.org/pdf/1801.01290.pdf)
