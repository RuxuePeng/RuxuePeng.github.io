---
layout: post
title:  "强化学习入门"
subtitle: Introduction to Reinforcement Learning Concepts
date:   2020-01-05 10:42:00
category: RL
tags: Reinforcement-Learning 中文文章
image: >-
  /assets/img/for_posts/P0/thumbnail1.JPG
optimized_image: >-
  /assets/img/for_posts/P0/thumbnail1.JPG

author: ruxuepeng
paginate: true
---

强化学习的本质就是学习如何作反应。强化学习系统通过与环境的互动，学会根据周围环境的状态作出合适的反应。这是它最接近人类学习方式的一点。  

## 本文要点

* 强化学习系统101    
* Evolutionary Method vs RL Method   
* 什么是Model-free System   

---

## 基本概念 - Concepts
* Agent - 智能体
* Interaction - （智能体与环境之间的）互动  
* Environment - 环境
* States - （环境的）状态
* Action - (某状态下可以采取的)行动
* Value - （某行动的）价值，也就是（基于某行动）能得到奖励的概率
* Value Function - 预测每个行动的价值的方程
* Policy - （某状态下）应该选择哪种行动的行为准则
* Evolutionary Method  
* Reinforcement Learning Method  
* Self-play  
* Exploration Moves  
* Greedy Moves    
* Model-free System  直接与真实环境交互的系统
    - 比如Q-learning, Sarsa, Policy Gradients
* Model-based System 尝试对环境建模的系统
    - 系统对所有能选的行动，利用模拟环境尝试预判出结果，并选择最优的那个
    - 比如送花还是送巧克力，系统预判送花的话、花粉过敏的女朋友会生病，预判送巧克力会加分，就会选择送巧克力
    - 而Model-free系统由于没有预判能力，可能会直接送花，然后被打


---

## 强化学习101     
与有监督学习(Supervised Learning)最大的不同是，强化学习的系统没有一个明确的“老师”(Supervisor)。它是通过不断尝试不断翻车的过程进步，也就是**Trial-and-error Search** .  
跟人一样，你现在做的选择可能不会对你的生活有立刻的影响，但是在未来的某一天，你发现这个决定其实改变了你人生的方向。这就是RL的第二个特点，奖励可能延迟发生(**Delayed Reward**) .  

---
### 1. 智能体(Agent）   
是强化学习系统的一个部分。一般满足这些要求:  
* 能感受到环境目前的状态(State)
* 能作出影响环境状态的行动(Action)  
* 有一个目标（Goal），而且这个目标跟它周围的环境状态相关  

比如，忘记给你过生日男朋友是智能体吗?  
不是，因为他不能感受到环境目前的状态.  

又比如，记得你生日但是没有给你过生日的男朋友是智能体吗?  
不是，因为他不能做出行动，或者他的目标跟环境这个状态不相关.  

咳咳言归正传...



---
### 2. thing2  

---
## 要点2

---

## 总结
placeholder

---
## Reference
* [Reinforcement Learning: An Introduction - 2nd Edition](https://web.stanford.edu/class/psych209/Readings/SuttonBartoIPRLBook2ndEd.pdf)
