---
title: DIALOGPT Large-Scale Generative Pre-trainingfor Conversational Response Generation
top: false
cover: false
toc: true
mathjax: true
date: 2021-02-07 14:27:23
password:
summary:
tags: [NLP , 论文]
categories: 论文阅读
---

## **《DialoGPT : Large-Scale Generative Pre-training for Conversational Response Generation》**

> 该论文发表在ACL-2020，是由Microsoft团队**Yizhe Zhang** 、**Siqi Sun**等作者共同完成的。

## 摘要

这篇论文提出了一个大型、可微调的神经对话响应生成模型，**DialoGPT**。是从Reddit中2005年至2017年的聊天对话数据中训练得来的。DialoGPT扩展了Huging Face PyTorch的transformer，使其在单论对话性能接近人类。作者表明，DialoGPT的会话系统比其他系统产生更加相关、内容更丰富以及上下文一致的响应。预先训练的模型和训练管道被公开发布，以促进神经响应生成的研究和开发更智能的开放域对话系统。



## 引言

基于transformer结构的大规模预训练模型已经取得了很大成功。例如OpenAI的GPT-2已经证明，在非常大的数据集上训练的transformer模型可以捕获文本数据中的长期依赖关系，并生成流畅、词汇多样和内容丰富的文本。这样的模型有能力捕获细粒度的文本数据，并以高分辨率产生输出，该高分辨率紧密模拟人类编写的真实世界文本。

**DialoGPT是在GPT-2的基础上拓展的**，用于解决对话神经响应生成的问题。神经响应生成是文本生成的一个子类别，但是它们都是生成与提示相关的自然文本。目前大多数开放域神经响应生成系统存在一些问题，**例如内容和风格不一致，缺乏长期的上下文信息和生成的响应过于平淡**。



## 数据集

该数据集是Reddit从2005年到2017年的评论链中提取的。**Reddit的讨论可以自然地扩展为树状结构的回复链**，将根节点到叶节点的每条路径作为包含多个对话回合的训练实例，并提取出来。我们通过删除其中的实例来过滤数据

- 源或目标中有一个URL。
- 其中目标包含至少三个单词的单词重复。
- 如果答复不包含至少一个最常见的50个英语单词(例如，“the”、“of”、“a”)，因为这可能表明它可能不是一个英语句子。
- 其中响应包含特殊标记，如“[”或“]”，因为这可能是标记语言。
- 其中源序列和目标序列一起超过200个单词。
- 其中目标包含攻击性语言，由与大块列表匹配的短语标识。

经过筛选，数据集包含147,116,725个对话实例，共计**18亿字**。



## 方法

### 模型结构

DialoGPT是在GPT-2结构的基础上训练的。GPT-2 的transformer模型采用的是通用的transformer语言模型，利用掩码多头自注意层所构成的栈来训前文本数据。我们的模型继承了GPT-2(Radford等人，2018年)，它由12至48层transformer并且包括normalization层，一种解释我们修改的模型深度的初始化方案，以及字节对编码用于标记器。

跟OpenAI的GPT-2一样，将多转对话会话建模为长文本，并将生成任务框架为语言建模。我们把源句子记作S，目标句子记作T，
$$
S= x_1, ...,x_m ， T = x_{m+1},...,x_N
$$
其中N是序列长度，因此条件概率P(T|S)可以分解为：
$$
p(T|S) = \prod_{n=m+1}^N p(x_n | x_1,...,x_{n-1})
$$
多轮对话T1,...,Tk，上式也可以写作$p(T_k,...,T_2 | T_1)$ ，它本质上是条件概率$P(T_i | T_1,...,T_{i-1})$的累乘。因此优化单个$p(T_k,...,T_2 | T_1)$ ，可以视为优化所有的$P(T_i | T_1,...,T_{i-1})$对。

### 最大互信息

开放域文本生成模型因生成平淡无奇的、缺乏信息的样本而臭名昭著。 为了解决这个问题，实现了一个最大互信息（**MMI**）评分函数。**MMI**采用一个预先训练的反向模型来预测给定响应的源句子，即P（source|target）。

首先使用top-K抽样生成一组假设。 然后利用P（source|Hypothesis）的概率对所有假设进行重新排序。 直觉上，最大限度地向后模型似然会惩罚平淡的假设。因为重复出现的假设可能与一些查询有关联，而对其他特定查询关联的概率很低。

本文还试图使用policy梯度与样本平均基线来优化奖励R=P（source|Hypothesis）。好的奖励可以稳定地让模型提高，但与RNN体系结构下的训练不同，我们观察到强化学习(RL)训练很容易收敛到退化的局部最优解。

## 结果

### 实验细节

训练了三个不同大小的模型，他们的参数分别是117M，345M和762M。模型使用了一个包含50，257个条目的词典，并且在16个Nvidia V100机器上训练，使用了NVLink。使用了Noam学习速率调度器和16000个热身步骤。学习率根据验证损失选择。 每个模型都被训练，直到验证损失没有更新。 对于中小型模型，我们训练了多达5个epochs的模型。 训练的大模型3个epochs。



**训练加速**：将训练数据压缩至lazy-loading 数据库文件。还利用单独的异步数据进程来扩展训练。除此之外，进一步使用动态批处理策略将长度相似的会话分组到同一批中，从而提高了训练吞吐量。

### 实验结果

基于DSTC的评价数据

![image-20210207163620461](https://i.loli.net/2021/02/07/czn2A4L1tlVXrmI.png)

--------

基于6K Reddit multi-reference 评价数据

![image-20210207163630238](https://i.loli.net/2021/02/07/DcXbMo7rxYpLR8w.png)

在表格3中，相较于Greedy 生成，MMI重排后会产生更加多样的响应。具有更好的NIST、METEOR、Entropy和DIst分数，但是BLEU分有所下降。

### 生成的例子

![image-20210207164127603](https://i.loli.net/2021/02/07/jlJgSbKiH9ZUd3W.png)

![image-20210207164133258](https://i.loli.net/2021/02/07/XD2kNIQfsTrRS56.png)

![image-20210207164234170](https://i.loli.net/2021/02/07/J9BwutckgbqW6Zs.png)

## 局限性和风险

尽管我们努力在训练前减少具有攻击性的数据，但DialoGPT依然有可能会产生不好的回应。输出的数据中隐含的性别偏见和其他历史偏见。发布DialoGPT的一个主要动机是使研究人员能够调查这些问题并制定缓解策略。 在任何情况下都不应因使用DialoGPT而产生不适当的内容被解释为反映作者或微软公司的观点或价值观。



**参考文献：**

1. Zhang Y, Sun S, Galley M, et al. Dialogpt: Large-scale generative pre-training for conversational response generation[J]. arXiv preprint arXiv:1911.00536, 2019.

