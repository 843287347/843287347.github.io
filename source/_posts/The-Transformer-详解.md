---
title: The Transformer 详解
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-20 14:03:44
password:
summary:
tags: [论文, NLP]
categories: 论文阅读
---

### 背景知识

对于构建语言模型和机器翻译等任务，使用基于Seq2Seq架构的RNN、LSTM、GRU等传统方法，已经取得了非常好的效果。尽管目前使用了一系列的方法，提升了RNNs的效率，例如使用因式分解，条件计算等技巧。但是RNNs根本的的缺陷并没有解决，还一直保留着。

注意力机制（Attention mechanisms）已经是序列建模（sequence  modeling）和传导模型（transduction model）的不可或缺的部分。它可以找到输入、输出序列之间的依赖程度，而不依赖于词与词之间的距离。

在论文中，提出了Transformer模型，它抛弃了循环神经网络，而是完全依赖于注意力机制的一种模型结构。



### 模型结构

目前效果好的神经序列传导模型都是基于encoder-decoder的结构。它的encoder部分是将符号序列表示$(x_1,...,x_n)$ 映射到连续的表示$z=(z_1,...,z_n)$，然后对于decoder部分，将$z$ 作为输入，最终输出符号序列$(y_1,...,y_m)$ ，在每一个时间步长都是自回归的：将$t-1$生成的输出，作为$t$的输入。

Transformer的模型结构如下所示：



![Transformer结构](https://i.loli.net/2021/03/20/EZduNkxml2DQ83c.png)

#### 编码器和解码器

**Encoder：** 这个编码器是由6个相同的层堆叠构成的，其中的每一层中包括两个子层。第一个子层是**多头注意力机制** ，第二个子层就是简单的**全连接前馈网络**。在每一个子层中都是用了**残差连接（residual connections）**和**层正则化（layer normalization）**。对于每一个子层，它的输出就相当于
$$
LayerNorm(x + SubLayer(x))
$$
同时为了方便，所有的输出维度都是$d_{model}=512$。



**Decoder：** 和编码器一样，解码器也是由6个相同的层堆叠构成的。对于每一个层，除了有相同的子层之外，还添加了第三个子层，它主要是是用于处理Encoder输出的序列。和Encoder一样也是使用了残差连接和层正则化。不同的是，修改了第一个子层的多头注意力机制，通过**掩盖**的方式，让解码器不能处理后面的内容。（例如，假如预测下一个词任务是，已知的信息只能是前面已经出现过的词，因此需要将多余的信息抹去）。



#### 注意力机制

**Scaled Dot-Product Attention**

注意力机制可以描述成，给定一个查询、键-值对映射到一个输出向量。在这里使用了特定了“Scaled Dot-Product Attention”注意力计算方法。它的计算公式是
$$
Attention(Q,K,V) = softmax(\frac{QK^T}{\sqrt{d_k}})V
$$
与additive attention 和dot-product 计算方法不同的是，它除以了$\sqrt{d_k}$，它可以很好地克服梯度很小的问题。

![Scaled Dot-Product Attention计算流程](https://i.loli.net/2021/03/20/tRqS1NehIHPmBn7.png)

**Multi-Head Attention**

多头注意力，就是通过$h$次线性投影，生成$h$个注意力层并分别计算权重。其中每一个注意力层都是通过Scaled Dot-Product Attention函数计算，最终将输出结果重新拼接到一起

![Multi-Head Attention](https://i.loli.net/2021/03/20/thM1qoXaFwcLQPE.png)

使用数学公式描述就是


$$
MultiHead(Q,K,V) = Concat(head_1,...,head_h)W^o
$$

$$
head_i =Attention(QW_i^Q,KW_i^K,VW_i^V)
$$

#### 基于位置的前馈神经网络

除了**多头自注意机制** ，在编码器和解码器的每一个层都包含了前馈全连接网络。它是由两个线性变化层和ReLU函数构成。
$$
FFN(x) =max(0,xW_1 + b_1)W_2 +b_2
$$


**位置编码**

因为循环神经网络可以很好的处理时序信息，而使用注意力机制的时候，并不能很好的提供位置信息，因此需要给添加一个位置编码。有多种位置编码的方法，在原论文中，使用了三角函数的位置编码：
$$
PE（pos,2i） = sin(pos/10000^{2i/d_{model}})
$$

$$
PE（pos,2i+1) = cos(pos/10000^{2i/d_{model}})
$$



$i$是位置编码的为第$i$个维度。