---
title: Hierarchical Question-Image Co-Attention for Visual Question Answering
date: 2021-05-18 15:25:46
tags:  [VQA, Attention]
categories: VQA
---

# 简介
<p style="text-indent:2em">
这篇文章提出了“协同注意力”机制，不只考虑视觉上的注意力（看哪儿），同时也考虑问题本身（重点词）。
另外，在图像和问题解析时构建“单词-短语-问题”三层分层结构。单词层使用嵌入矩阵将单词转化成词向量；
短语层使用1维核卷积获取词组中包含的信息，通过将词表征向量和不同时序过滤器做卷积，然后将多个
n-元词组通过池化得到一个短语层的表征向量；在问题层，使用循环神经网络编码整个问题。
</p>

# 相关工作
<p style="text-indent:2em">
</p>

## 图注意力
- spatial attention
	在标准LSTM模型中添加空间注意力机制
- compositional scheme
	由文本解析器和多个神经网络模块组成，由解析器决定调用哪个模块解答问题，模块化网络参考论文
    [Deep Compositional Question Answering with Neural Module Networks](https://arxiv.org/abs/1511.02799)
- stacked attention network
	多跳图像注意力框架;先将单词映射到图像区域，再通过注意力机制选择相关区域形成解答
- dynamic memory 
	使用新的输入融合模型和基于GRU的注意力机制得到答案

## 语言注意力
<p style="text-indent:2em">
在VQA领域还没有相关的工作，不过在NLP领域对此有较多研究。
</p>

- Neural machine translation by jointly learning to align and translate
- Reasoning about entailment with neural attention
- Teaching machines to read and comprehend
- Abcnn: Attention-based convolutional neural network for modeling sentence pairs
- Attentive pooling networks

# 研究方法

## 符号介绍

包含 $T$ 个字符的问题的向量表示记为 $Q=\lbrace q_{1}, ... , q_{t} \rbrace$, $q_{t}$ 表示第 $t$ 个词的特征向量。
$q_{t}^{w}$,$q_{t}^{p}$,$q_{t}^{s}$分别表示 $t$ 位置的词嵌入，短语嵌入和问题嵌入向量。图像特征记为 
$V = \lbrace \nu_{1}, ..., \nu_{N} \rbrace$,$\nu_{n}$是空间位置$n$的特征向量。每层的图像和问题的协同注意力特征
记为$\hat{\nu}^{r}$和$\hat{q}^{r}$, $r \in \lbrace w, p, s \rbrace$。

## 问题分层
将问题中的单词用$1-hot$编码表示，先将单词编码映射到单词层向量$Q^{w}$。短语层的向量使用下式表示：
$$\hat{q}_{s,t}^{p} = tanh(W_{c}^{s}q_{t:t+s-1}^{w}), s \in \lbrace 1, 2, 3 \rbrace$$
其中$W_{c}^{s}$为权重参数，$t$位置的短语层特征取
$$q_{t}^{p} = max(\hat{q}_{1,t}^{p}, \hat{q}_{2,t}^{p}, \hat{q}_{3,t}^{p}), t \in \lbrace 1, 2, ..., T \rbrace$$

## 协同注意力机制
文章提供了两种生成图像和问题映射的协同注意力机制。

<div>
<img src="https://d3i71xaburhd42.cloudfront.net/fb9d253258d6b3beceb9d6cd7bba6e0a29ab875b/250px/4-Figure2-1.png" width="70%" />
Figure 2: (a) Parallel co-attention mechanism; (b) Alternating co-attention mechanism. 
</div>

### Parallel Co-Attention
Parallel Co-Attention通过计算image和question特征之间的相似性，使image和question联系起来，相关矩阵的计算方法：
$$C = tanh(Q^{T}W_{b}V)$$
把$C$当成一种特征，预测image和question的attention maps。

$$H^{v} = tanh(W_{v}+(W_{q}Q)C), H^{q} = tanh(W_{q}Q + (W_{v}V)C^{T})$$
$$a^{\nu} = softmax(\omega_{hv}^{T}H^{\nu}), a^{q} = softmax(\omega_{hq}^{T}H^{q})$$
$$\hat{v} = \sum_{n=1}^{N}{a_{n}^{v}v_{n}}, \hat{q} = \sum_{t=1}^{T}{a_{t}^{q}v_{t}}$$

### Alternating Co-Attention
整个过程分为三部分：
1) 将问题映射成一个单向量$q$
2) 基于$q$，集中注意于image
3) 基于attended image feature, 集中注意question

注意力的操作部署如下：
$$H = tanh(W_{x}X + (W_{g}g)\mathbb{1}^{T})$$
$$a^{x} = softmax(w_{hx}^{T}H)$$
$$\hat{x} = \suma_{i}^{x}x_{i}$$

第一步： X = Q, and g is 0;
第二步： X = V where V is the image features，guidance g is intermediate attended question feature ^s from the first step
第三步： we use the attended image feature ^v as the guidance to attend the question again, i.e., X = Q and g = ^v.

### 预测答案编码
文章将VQA视为分类任务，基于图像和问题的协同注意力特征预测答案。

<div>
<img src="https://d3i71xaburhd42.cloudfront.net/fb9d253258d6b3beceb9d6cd7bba6e0a29ab875b/5-Figure3-1.png" width="70%" />
Figure 3: (a) Hierarchical question encoding; (b) Encoding for predicting answers
</div>

按上图右所示，使用MLP循环编码注意力特征得到最终答案。

$$h^{w} = tanh(W_{w}(\hat{q}^{w} + \hat{v}^{w}))$$
$$h^{p} = tanh(W_{p}[(\hat{q}^{p} + \hat{v}^{p}), h^{w}])$$
$$h^{s} = tanh(W_{s}[(\hat{q}^{s} + \hat{v}^{w}), h^{p}])$$
$$p = softmax(W_{h}h^{s})$$
