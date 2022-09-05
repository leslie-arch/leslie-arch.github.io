---
title: VGQ综述
date: 2021-05-22 17:56:59
tags: "VQG"
categories:
- ["VQG"]
- ["VQA"]
---

# VQG 综述

论文title [Visual Question Generation: The State of the Art](https://www.webofscience.com/wos/alldb/full-record/WOS:000582586400002)

Question generation : 以原始文本，数据库和语义表示等为输入自动生成问题。
其中最重要的两个子任务：
- 理解图像
  - 对象检测
  - 对象分类和标签
  - 对象关系检测
  - 场景理解
  - 场景分类
  - 其它
- 生成有效问题的词序列

通常首先通过CNN获取图像的表示，再通过RNN获取生成的词序列。

VQA包括在给定的图像上下文中回答给定的问题,几乎所有用于此目的的VQA数据集都包含人工预先设定的问题，最近新生成的推理数据集却与此不同，
其中的问题及相应的答案是使用功能性编程方法获得的。

# 视觉和自然语言任务

视觉问题可分为三类：
- 基础问题 :: 问题答案可由图像本身中存在的信息得出，不需要外部知识来源
- 基于常识的问题 :: 需要外部的常识知识源以及图像中的基础信息
- world knowledge-based question :: 需要外部知识源用于问题生成和回答

所有VQA算法的框架流程都包含：
- 获取图像特征
- 抽取问题特征
- 结合问题特征和图像特征预测问题答案

按过程可分为四类： Bayesian， encoder-decover, attention based, bilinear pooling
同时，使用一些其它外部知识源回答视觉问题的领域也有了新的发展。

## Bayesian Models
VQA最早是在Bayesian框架中进行问题推理进行自动语义文本解析和场景分割。Bayesian过程可以对问题进行分类，从而检测答案类型。
该模型证明了开放式答案预测可以转化为多项选择问题。

## Encoder-Decoder Models
Cho 和 Sutskever 提出的简单的序列到序列机器翻译 encoder-decoder模型被认为是语言生成任务中的突破，对图像和问题信息进行编码然后对其解码从而得到答案的灵感也来源于此。
基于单LSTM的编/解码器模型依次将问题和答案的单词分别进行编/解码，通过CNN提取的图像特征作为每个LSTM单元的输入。
[Ren et al](https://arxiv.org/abs/1505.02074)提出在回答是单个单词的情况下，将<问题,回答>特征向量对连接作为LSTM的输入。在ImageNet挑战的训练过程中使用VGGNet
最后一层隐藏层作为视觉嵌入表示。
[Gao et al](https://arxiv.org/abs/1505.05612)提出的模型使用两个独立的LSTM，一个用于问题编码，一个用于回答解码。回答生成的融合模块使用深度CNN整合问题编码表示，
回答编码表示和图像表示。

## 注意力模型
所有注意力模型都使用由CNN生成的区域特异性和局部特征。
[Shih et al.](https://arxiv.org/abs/1511.07394)基于注意力的图像区域和问题表示给每个回答打分，在多个候选解答中的预测正确回答。

## bilinerar pooling 模型
图像和问题特征之间深层交互的紧凑型多模双线性池化。
bilinear pooling主要用于特征融合，对于从同一个样本提取出来的特征 $x$ 和特征 $y$ ，通过bilinear pooling得到两个特征融合后的向量，进而用来分类。
如果特征 $x$ 和特征 $y$ 来自两个特征提取器，则被称为多模双线性池化（MBP，Multimodal Bilinear Pooling）；
如果特征 $x$ =特征 $y$，则被称为同源双线性池化（HBP，Homogeneous Bilinear Pooling）或者二阶池化（Second-order Pooling）。

## 结合外部知识的VQA

[Wu et al.](https://arxiv.org/abs/1511.06973)使用VGGNet获取图像的高层概念和属性，然后使用这些特征生成SPARQL查询请求从知识库中抽取相关信息。
knowledge-based VQA,视觉概念分为三种，为每张图片标识对象，属性和场景。通常使用RDF元组存储这些概念。
fact-based VQA, 通过学习问题和基于知识的问题的关联回答组合的问题。从图片中抽取的视觉概念以$(subject, relation, object)$元组的方式存储。使用元组生成SPARQL的查询
从知识图谱中抽取问题答案。
world knowledge VQA，结合其背景知识回答特定人物的相关问题。该问题的第一个子任务是可视化实体链接，处理人脸定位和人脸识别。第二个子任务需要识别图片和问题中实体，
然后遍历知识图，如Wikidata，来抽取相关事实。

# 图像描述生成
和VQG一样，图像描述任务也是以图片为输入，期望输出为文本形态。可将其视为一种特殊的机器翻译，将视觉信息翻译成文本格式。
图像加注方法可以分为3类：基于模板的图像加注，基于检索的图像加注和生成新的图像加注。
新的图像加注与生成新的问题方法类似。

Multi-model space embedding:
[多模态空间嵌入模型](https://www.aclweb.org/anthology/Q14-1017.pdf)通过AlexNet抽取图像特征用于图片加注，在这基础上扩展的encoder-decoder模型中，通过深度卷积网络得到的图像特征作用于LSTM隐层状态生成编码。
DT-RNN基于dependency tree结合图像和文本的嵌入空间。
m-RNN模型包含三个主要部分：语言模型部分，视觉部分（CNN），多模态部分用于融合前两部分的结果，然后生成标注。

encoder-decoder模型:
- neural image caption generator (NIC)
- Bi-LSTM :: 生成上下文和语义丰富的图像标注

Attention模型：
- deterministic soft attention
- stochastic hard attention

Dense Captioning 模型:
[Densecap model](https://arxiv.org/pdf/1511.07571.pdf)提出"Fully Convolutional Localization Network"，由一个卷积网络，一种新的全连接定位层和一个产生标签序列的RNN语言模型组成。

- 模型能够描述图片中的一系列区域，而不是单个object
- 全连接定位层，预测图片中的感兴趣区域，并使用双线性插值提取区域中的激活值

Compositional 模型:
[Compositional](https://arxiv.org/abs/1411.4952)的思路：
- 给出图像，使用视觉检测器提取图像中可能存在的单词(Weakly-spuervised approach of multiple instance learning) & MIL(Multiple Instance Learning)
- 采用传统的方法得到若干预测的captions
- 使用DMSM（Deep Multimodal Similarity Model）模型计算相似度

Generative 模型：基于生成网络的图像加注模型，可以生成大是不是的图像标（个人对这种模型不看好）

Captioning With External Knowledge：
[Rich Image Caption](https://arxiv.org/abs/1603.09016)，针对特定场景的图像，结合知识图谱生成标：
- ResNet :: visual concept
- Entity Recognition Model
- Maxmum Entropy Language Model
- DMSM (模型相似度)

# Text Question Generation and Answering

Inverse VQA:
VQA是根据图像和问题生成答案，iVQA是给出answer,image, 生成与之相关的question。
过程与VQA过程相似，用到的模型也是Bi-LSTM, GUR和encoder-decoder模型。

Using Memory Network:
Dynamic memory network - 包含四个部分：输入模块，问题模块，episodic memory 模块和回答模块。
输入模块/问题模块分别生成输入文本和问题并且生成它们各自的向量表示。episodic memory模块实现了注意力机制并选择要关注的输入部分。迭代过程用问题和上轮迭代的内存表示更新内存向量。
在DMN的基础上进行扩展得到的新模型[DMN+](https://arxiv.org/abs/1603.01417)。

Using External Knowledge Source:
- [key-value memory network based model](https://arxiv.org/abs/1704.08384)
- [Factoid question generation model](https://arxiv.org/abs/1603.06807)
- [RNN-based encoder-decoder model](https://www.aclweb.org/anthology/E17-1036/)


# 相关的数据集
- DAQUAQ
- COCO QA
- VQA 
- VQG Flickr-5000, VQG COCO-5000, and VQG Bing-5000
- Visual Genome
- Visual7W
- CLEVR
- FVQA/KVQA/GQA
- CRIC
