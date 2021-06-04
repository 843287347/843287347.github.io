---
title: BERT 详解
top: true
cover: true
toc: true
mathjax: true
date: 2021-03-20 21:07:08
password:
summary:
tags: [NLP, 论文]
categories: 论文阅读
---

[1].Devlin J, Chang M W, Lee K, et al. Bert: Pre-training of deep bidirectional transformers for language understanding[J]. arXiv preprint arXiv:1810.04805, 2018.

### 背景知识

BERT（**B**idirectional **E**ncoder **R**epresentations from **T**ransformers），是一个强大的预训练模型，它是基于对无标签文本**联合学习**训练得到的。

目前有两种策略将预训练语言模型到下游任务中，分别是*feature-based* 和 *fine-tuning*。第一种方法，例如ELMo模型，它将预训练的表示作为额外的特征加入到模型里。第二种方法，例如GPT模型，它引入了模型的参数，然后通过简单的微调，得到好的模型。

但是以上两种方法都只使用了单向的语言模型，去学习通用的语言表示，这严重地限制了预训练表示的能力，因此提出了BERT模型。

![BERT的预训练和微调](https://i.loli.net/2021/03/20/DEQYKgpAB2MCofb.png)

### 预训练BERT

在训练BERT的时候使用了两个无监督任务，分别是掩码语言模型（Masked LM）和下句预测（Next Sentence Prediction NSP）

#### **Masked LM**

为了训练深度双向表示，通过随机的将一定比例的词进行遮盖，然后让模型去预测这个词。对于训练的数据中，随机选取15%的词，对于每一个选取的词，有80%的概率替换成[MASK] ，10%的概率不会变化，10%的概率用其他词替换。

#### **Next Sentence Prediction**

像问答系统（QA）和自然语言推理（NLI）任务，主要是要模型去理解两个句子之间的关系。因此对于输入的两个句子A和B，其中有50%的概率是A是真正的下一句，有50%的概率是随机的句子。

> **Input** = [CLS] the man went to [MASK] store [SEP]  he bought a gallon [MASK] milk [SEP]
>
> **Label** = IsNext
>
> 



> **Input** = [CLS] the man [MASK] to the stroe [SEP] penguin [MASK] are flight ## less birds [SEP]
>
> **Label** = NotNext

### 微调BERT

微调就是直接干脆地，因为Transformer的自注意机制让BERT可以对不同的下游任务进行建模，通过选择合适的输入和输出的格式。对于一些常见的任务，可以选择

- Sentence-Paraphrasing 句子-文章对
- Hypothesis-Premise 假设-前提对
- Question-Passage 问题-段落对
- 退化的 Text - null 对

#### BERT的模型结构

![不同预训练模型结构的差别](https://i.loli.net/2021/03/20/wEaXpLFoSA5rUyR.png)

我们可以看到OpenAI 的GPT模型使用的是从左向右的Transformer，ELMo使用的是级联的双向LSTMs结构。而BERT则是结合了双向的文本信息，通过使用Transformer块。