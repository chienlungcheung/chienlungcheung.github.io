---
title: "自然语言的可计算性：从 N-gram 到 BERT"
date: 2023-11-19T12:11:15+08:00
draft: false
---

[toc]

语言符号化是逻辑计算的前提，最近这一百年，从图灵、丘奇到麦卡锡、闵斯基再到科尔莫格洛夫，这些前辈都在寻找语言的可计算性。最近这几十年，语言的可计算性经历了从 N-gram 到 word embedding 再到 attention 演进，每一次的突破都将 NLP 领域推进了一大步。本文试图来描述这个演进过程。

## 1 语言模型(LM)技术体系的时代划分

针对语言模型技术体系的演进，我们可以依据关键技术的出现和发展来进行分段。

- 古典时代（Classical Era）：
  ◦ 起始：早期的统计方法，如基于词频的方法。
  ◦ 高潮：N-gram 模型的发展。
  ◦ 结束：N-gram 模型因数据稀疏性问题而受到限制。
  ◦ 特点：主要依赖统计和计数方法，模型简单，容易理解。
- 嵌入时代（Embedding Era）：
  ◦ 起始：神经网络语言模型的初步尝试，如 Bengio 于 2003 年的论文（[《A Neural Probabilistic Language Model》](https://www.jmlr.org/papers/volume3/bengio03a/bengio03a.pdf)）。
  ◦ 高潮：Word2Vec、GloVe 等词嵌入技术的诞生和普及。
  ◦ 结束：嵌入技术成为标准，但**缺乏长距离上下文信息捕获**能力。
  ◦ 特点：词向量能够捕获语义和句法信息，但模型通常只考虑局部上下文。
- 深度学习时代（Deep Learning Era）：
  ◦ 起始：RNN、LSTM 和 GRU 的广泛应用于语言建模。
  ◦ 高潮：Transformer 架构和 GPT、BERT 等预训练模型的成功。
  ◦ 现状：巨大的预训练模型如 OpenAI 的 GPT-3 和 Google 的 BERT 系列继续主导这个领域。
  ◦ 特点：能够处理长距离上下文，模型复杂度增加，需要大量的数据和计算资源。

每个时代都有其独特的特点和挑战。

N-gram 是自然语言处理和语言建模中的经典技术，以下是 N-gram 解决的问题和遗留的问题：

1. 解决的问题：
- 简单性与效率：N-gram 模型相对简单，它只是基于统计计数来预测下一个单词的出现。因此，构建和使用 N-gram 模型是高效的，特别是对于大规模的文本数据。
- 本地上下文捕捉：N-gram 模型可以捕捉到 n-1 个单词的局部上下文，这在很多应用中是有用的，比如拼写检查和一些简单的文本生成任务。
• 无需复杂训练：N-gram 模型不需要像神经网络模型那样的迭代训练过程，只需一次通过数据进行计数。
• 模型可解释性：与深度学习模型相比，N-gram 模型更容易理解和解释，因为它基于实际文本中的统计计数。
2. 遗留的问题：
• 稀疏性问题：尽管 N-gram 可以捕获有限的上下文，但随着 n 的增大，数据中出现的唯一的 N-gram 组合会急剧增加，导致数据稀疏问题。这意味着很多 N-gram 组合可能不会在训练数据中出现，但在实际应用中可能会遇到。 考虑一个极端情况：对于一个包含 10,000 个词的语料库，2-gram 的可能组合是 $10,000^2$，而 5-gram 的可能组合是 $10,000^5$。实际语料库中的这些组合只是这些可能组合的一小部分。这意味着许多组合可能在训练数据中只出现一次或根本不出现。这就导致了数据稀疏问题：模型对于许多 N-gram 组合可能没有足够的数据来准确估计它们的概率。
• 固定窗口大小：N-gram 模型的窗口大小是固定的，这意味着它不能捕获超出固定窗口大小的长范围依赖。
• 缺乏语义理解：N-gram 模型完全基于统计计数，它没有捕捉到单词间的语义关系。例如，“猫”和“猫科动物”在语义上可能很相似，但 N-gram 模型不能识别这种关系，除非它们在相同的上下文中经常出现。
• 计算和存储需求：随着 n 的增大，需要存储的 N-gram 组合数量也会增加，这可能导致存储和计算上的挑战。
• 平滑问题：为了解决不在训练集中出现的 N-gram 组合问题，需要使用各种平滑技术，如 [Add-k 平滑](https://courses.engr.illinois.edu/cs447/fa2018/Slides/Lecture04.pdf)。这些技术本身带有一些偏见和限制。
• 不灵活：N-gram 模型通常对于其训练数据外的新的、不同的上下文是不灵活的。

总的来说，尽管 N-gram 为一些 NLP 问题提供了简单有效的解决方案，但其固有的限制也促使研究者探索新的、更复杂的模型，如基于神经网络的语言模型，**以捕捉语言的复杂性和多样性**。

从 Bengio 于 2003 年提出的基于神经网络的语言模型开始，Word embedding 技术经历了几个关键的发展阶段：

• Bengio的神经网络语言模型 (2003):
    ◦ 解决的问题：传统的 N-gram 模型遭受了稀疏性的问题，尤其是当词汇量很大或需要考虑更长的依赖时。Bengio 的模型使用了一个前馈神经网络来学习单词的分布式表示，并用这些表示来预测下一个单词，从而解决了稀疏性的问题。
    ◦ 遗留的问题：尽管该模型成功地引入了神经网络到语言建模中，但它的训练仍然相对缓慢，并且计算成本较高。
• Word2Vec (Mikolov 等人, 2013):
    ◦ 解决的问题：Word2Vec 提出了两种有效的模型架构（CBOW 和 Skip-gram），这两种架构都大大提高了单词嵌入的训练速度和效率。Word2Vec的嵌入捕获了单词之间的语义和句法关系。
    ◦ 遗留的问题：每个单词只有一个嵌入，这意味着词义多义性的问题（即一个单词有多个含义）没有被直接处理。
• GloVe (Pennington等人, 2014):
    ◦ 解决的问题：GloVe模型是基于全局统计信息的，而不仅仅是局部信息。这使得嵌入更能反映大量语料库中的统计规律。
    ◦ 遗留的问题：与Word2Vec相同，每个单词只有一个嵌入，没有处理多义词问题。
• ELMo (Peters 等人, 2018):
    ◦ 解决的问题：ELMo 使用了深度双向 LSTM 来获取上下文敏感的单词嵌入，从而能够为一个单词生成不同的嵌入，取决于它的上下文，有效地处理了词义多义性问题。
    ◦ 遗留的问题：尽管 ELMo 嵌入提供了丰富的语义信息，但计算成本仍然较高，特别是对于需要实时处理的应用。
• Transformer & BERT (Vaswani 等人, 2017; Devlin 等人, 2018):
    ◦ 解决的问题：Transformer 结构引入了自注意力机制，使模型能够并行化处理整个句子，大大提高了效率。BERT 则利用了 Transformer 的结构，并通过双向无监督预训练进一步增强了上下文敏感性。
    ◦ 遗留的问题：虽然 BERT 的性能非常出色，但它的模型尺寸巨大，对计算资源的需求也很高。

自从 Bengio 的初步工作以来，Word embedding 技术已经取得了长足的进步，特别是在捕获单词和上下文之间的复杂关系方面。每一步的进展都在解决前一步的限制，但同时也带来了新的挑战，这推动了研究的进一步发展。

由于嵌入时代与大模型诞生密不可分，下面重点讲一下这个时代及其之后的技术演进。

## 2 CNN：图像预训练的启示

![Alt text](image.png)

自从深度学习爆火，预训练过程就成为图像领域的常规方法了，而且这种做法很有效，能明显促进应用的效果。

![参考：https://pyimagesearch.com/2019/06/03/fine-tuning-with-keras-and-deep-learning/](image-1.png)

上图展示图像领域的预训练过程。我们设计好网络结构以后，对于图像来说一般是 CNN 的多层叠加网络结构。可以先用某个训练集合（一般是通用型数据，如 imagenet）对这个网络进行预训练，在对应任务上学会网络参数，然后存起来以备后用。假设我们面临另一个任务，网络结构采取相同的网络结构。在较低的 CNN 层，网络参数初始化的时候可以加载前述在通用数据上学习好的参数；在较高的 CNN 层，参数仍然随机初始化。参数设置好后用新任务的数据来训练网络。

在新任务上训练数据时，针对参数迭代有两种做法。一种是“frozen”（参数冻结），即之前从通用任务引入的浅层参数在训练新任务过程中不变;另外一种是“Fine-Tuning”（参数微调），即全量参数在新任务训练过程中会被改变，但这种改变是在预训练参数技术上进行的相比从零训练全量参数要简单很多。不管采用哪种做法都有一个很大的好处，即使得过去因为数据量太少而无法训练的任务变得可达。

针对动辄上亿参数的网络结构如 Resnet、Densenet 等，少量的训练数据很难训练好如此复杂的网络。但是如果其中大量参数预先通过大的训练集合（如 ImageNet）进行训练，然后将训练好的参数直接拿来初始化新任务的大部分网络结构参数，再在新任务比较少的数据量上进行训练，就容易多了。此外，即使新任务可用训练数据量很大，前置预训练过程也能极大加快新任务训练的收敛速度。

预训练为什么可行？

CNN 的设计在某种程度上受到了动物视觉系统的启发，尤其是猫和猴子的视觉皮层。在 20 世纪 60 年代，两位神经生物学家 David Hubel 和 Torsten Wiesel 对猫的视觉皮层进行了实验，发现存在专门对特定方向的边缘或线条响应的神经元，这与 CNN 中的边缘检测器有相似之处。

![参考：https://medium.com/analytics-vidhya/the-world-through-the-eyes-of-cnn-5a52c034dbeb](image-2.png)

CNN 的层级结构使其能从浅到深抽象不同层次的特征：
• 浅层（例如第一、第二层）： 主要捕捉基本的视觉特征，如边缘、颜色和纹理。这些特征通常是局部的和简单的。
• 中层： 开始捕捉更复杂的特征，如简单的形状和模式，例如圆圈、条纹等。
• 深层： 能够表示更高级、更抽象的特征，如物体的各种部分、复杂的形状等。
• 最深层： 在更高的抽象层面，能够识别整个物体或场景，如狗、车或风景等。

越是底层的特征越通用，也就是说不论什么领域的图像都会具备诸如边角线弧线等底层基础特征；层级越高，抽取出的特征越与具体任务相关。因此，预训练好的网络参数，越是底层的网络参数抽取出特征越与具体任务无关。这种通用性正是预训练为什么可行的原因。因此，如果有预训练好的通用模型，在针对新的具体任务做训练之前，可以使用预训练模型底层参数初始化新任务网络参数。由于高层特征跟具体任务关联较大，可以随机初始化或者采用 Fine-tuning 方式通过新数据集“清洗”掉高层无关的参数。

既然预训练方法在图像领域这么好用，那 NLP 领域是否可以做类似的尝试呢？

## 3 Word2Vec：自然语言嵌入神经网络

Word embedding 诞生于 2003 年，我们可以将它看作 NLP 领域的预训练技术。

介绍 word embedding 之前需要先了解语言的分布式表示和语言模型：

1. 语言的分布式表示（Distributed Representation）：
- 这是一种表示文本信息（如单词、短语、句子等）的方法，通常采用稠密的向量形式。
- 分布式表示的核心观念是：词义由其在文本中的上下文定义。换句话说，出现在相似上下文中的单词应该有相似的表示。因此，"cat" 和 "kitten" 这两个单词的向量表示会比 "cat" 和 "car" 更加接近。
- Word2Vec、GloVe 和 FastText 是常用的方法来得到单词的分布式表示。
2. 语言模型（Language Model，简称 LM）：
- 语言模型是用来预测一个词序列的概率的模型。例如，给定一个句子或文本片段的前半部分，它可以预测下一个词是什么。具体来说，给定一个词序列 *w1,w2,…,wn*，语言模型的任务就是评估这个词序列出现的概率 $P(w_1,w_2,…,w_n)$。
- 常见的语言模型有：
  - 统计语言模型如 N-gram 模型：这是最简单的语言模型之一，它使用词的 n-1 个前面的词作为上下文来预测当前词。
  - 神经网络语言模型如循环神经网络（RNN）模型：RNN 可以捕捉长距离的依赖关系，用于建立更加复杂的语言模型。
  - 基于注意力的语言模型如 Transformer 和基于 Transformer 的模型：这类模型近年来非常流行，特别是 BERT、GPT、T5 等。它们使用 self-attention 机制来捕获文本中的长距离和短距离依赖。
3. 两者联系与区别：
- 当我们训练深度学习语言模型（如基于 Transformer 的模型）时，模型的内部会为输入的文本自动学习一种分布式表示。这些表示捕捉了文本中的语义和语法信息，并可以用于其他 NLP 任务。例如，BERT 是一个预训练的语言模型，它通过遮蔽（masked）单词来进行训练。当训练完成后，即使我们不使用 BERT 作为语言模型，我们仍然可以使用其内部的分布式表示来完成其他任务，如文本分类、情感分析等。
- 分布式表示和语言模型是两个相关但不同的概念，前者是文本信息的表示方式，后者是预测词序列概率的模型。但在深度学习的语言模型中，两者往往紧密结合。

无论是基于统计的语言模型还是基于神经网络的语言模型，它们的**核心目标都是最大化一个序列对应的联合概率**。但是，为了在实际应用中避免计算困难和解决数据稀疏性问题，这些模型通常都会利用链式规则将联合概率分解为条件概率的乘积。例如：
- n-gram 语言模型：这是一个统计方法，它使用 n 个词的滑动窗口来估计条件概率。n-gram 模型的限制在于它只能捕获 n 个词的局部依赖关系，而且对于稀有的 n-gram 组合，可能没有足够的数据进行准确估计。
- 神经网络语言模型：这种模型**使用神经网络来估计条件概率**。由于神经网络可以学习分布式表示，它能够捕获更长距离的依赖关系，并且更好地处理数据稀疏性问题。例如，word2vec、LSTM 和 Transformer（如 GPT 和 BERT）等模型都属于这类。

在训练这些模型时，无论是统计模型还是神经网络模型，它们都会试图最大化整个数据集上的条件概率，从而间接地最大化联合概率。

那我们如何设计一个神经网络来实现一个语言模型呢？我们期望针对训练好的神经网络，输入一句话前面若干单词，它能正确输出后面紧跟的单词。

![参考：《A Neural Probabilistic Language Model》https://www.jmlr.org/papers/volume3/bengio03a/bengio03a.pdf](image-3.png)

我们可以设计如上图的网络结构，这其实就是大名鼎鼎的“神经网络语言模型”。2003 年，Bengio 发表在 JMLR 上的论文《A Neural Probabilistic Language Model》创造了该模型。2013 年，当深度学习开始渗透到 NLP 领域的时候，该设计重新焕发光彩。

下面我们来看如何设计并实现它。

神经网络的学习目标为输入某个单词如 *model* 的前面 n-1 个单词，要求网络正确预测单词 *model*，即最大化条件概率 $P(W_t="model"|W_{t-n+1},W_{t-n+2},...,W_{t-1};θ)$，其中 θ 是前述条件概率最大时的模型参数取值。

训练过程为：
1. $W_t$ 前面每个单词 $W_i$  都用 Onehot 方式编码（比如 0001000）作为原始单词输入。
2. 前述每个 onehot 向量分别乘以矩阵 C 后获得向量 $C(W_i)$，该向量即为 $W_i$ 对应的 word embedding。
3. 然后将每个 $C(W_i)$ 级联在一起，输入后续的隐层。
4. 最后通过 softmax 去预测后面应该接哪个单词。

我们重点谈一下 $C(W_i)$ 的学习过程。

![Alt text](image-17.png)

前面提到 $C(W_i)$  就是单词对应的 Word Embedding 值。矩阵 $C$ 包含 $V$ 行，$V$ 代表词典的大小，每一行内容代表对应单词的 Word embedding 值。当然，矩阵 $C$ 需要通过学习获得的网络参数；训练初试可以用随机值初始化矩阵 $C$；当这个网络训练好后，矩阵 $C$ 的内容就会被正确赋值，每一行代表一个单词对应的 Word embedding 值。因此，通过这个神经网络学习任务，我们不仅自己能够根据上文预测后接单词是什么，同时获得一个副产品矩阵 $C$，即每个单词的 Word Embedding。

2013 年，NLP 领域最振奋人心的工作就是 Word2Vec，它可以更好地计算出 word embedding。Word2Vec 工作原理如下图：

![Alt text](image-4.png)

Word2Vec 的网络结构与前述 Bengio 设计的网络结构基本类似。不过这里需要指出：尽管网络结构相近，而且也是做语言模型任务，但是其训练方法不太一样。

Word2Vec 有两种训练方法。第一种叫 CBOW，其核心思想是从一个句子里面把一个词抠掉，用这个词的上文和下文去预测被抠掉的这个词（后面 BERT 的掩码语言模型 MLM 与之异曲同工）；第二种叫做 Skip-gram，和 CBOW 正好反过来，输入某个单词，要求网络预测它的上下文单词。这两种方法与前述 Bengio 设计的网络仅用一个单词的上文预测该单词的做法是有显著差异的。

为什么 Word2Vec 这么处理？原因很简单，Bengio 设计的网络主要任务是要学习一个解决语言模型任务的网络结构，语言模型就是要看到上文预测下文，而word embedding 部分只是该设计的一个副产品；而 Word2Vec 目标不一样，它的目标就获取更好地 word embedding。

Word2Vec 发明的 CBOW 的训练方法与后来的 BERT 关系莫大，同时它也是典型的预训练过程，可以在其之后接上具体的语言模型任务，如下图所示。

![Alt text](image-5.png)

它的使用方法与前面 Bengio 设计的网络是一样的：句子中每个单词以 Onehot 形式作为输入，每个 onehot 向量乘以学好的 Word Embedding 矩阵 $C$，获得单词对应的 Word Embedding。

不知道大家发现没有，上面这个操作本质是个查表操作。当我们用词典排序为 $i$ 的单词对应的 onehot 向量（$1$ 行 $V$ 列，且仅第 $i$ 个值为 $1$）乘以矩阵 $C$ （$V$ 行 $M$ 列）的时候，得到的就是矩阵 $C$ 的第 $i$ 行；由此可见，前述向量矩阵相乘操作可以避免，直接取出矩阵 $C$ 的第 $i$ 行即可。

我们现在回到预训练话题。

Word Embedding 矩阵 $C$ 其实就是 Onehot 输入层到 embedding 层的网络参数。因此，针对新任务使用训练好的 Word Embedding 等价于将新任务对应网络的 Onehot 层到 embedding 层的网络参数初始化了。这与图像领域使用通用任务的低层参数初始化新任务对应的低层参数是一样的。区别在于 Word Embedding 只能初始化第一层网络参数，再高层的参数就无能为力了。下游 NLP 任务在使用 Word Embedding 的时候也类似图像有两种做法，一种是 Frozen，就是 Word Embedding 那层网络参数固定不动；另外一种是 Fine-Tuning，就是 Word Embedding 这层参数使用新的训练集合训练也需要跟着训练过程更新掉。

Bengio 和后来的 word2vec 开启了 NLP 领域的预训练，但是这时候 Word Embedding 还存在一些缺陷。典型地，基于 word2vec 技术，每个单词只有一个向量，但是日常使用中经常出现一词多义的情况。这个问题影响了 word embedding 在实际案例中的应用，亟待解决。

## 4 ELMo：双向模型三层嵌入消除歧义

针对 word2vec 遗留的一词多义问题，ELMo 提供了一种简洁优雅的解决方案。ELMo 是“Embedding from Language Models”的简称，出自论文《Deep contextualized word representations》。ELMo 与以往方案差异最大的点即论文标题中提到的“deep contextualized”。ELMo 之前的做法都是将预训练模型最后一层（top layer）的输出串接到下游模型上，而 ELMo 是将自身网络各层参数做了线性组合后再输入到下游模型中。

学习高质量的单词表示是一个很大的挑战，ELMo 将问题具体建模为：1、如何学习单词复杂特征如语法和语义；2、如何处理一词多义。

个人认为，ELMo 考虑了语言双向训练的重要性， 先进性堪比后来的 BERT。

### 4.1 ELMo 预训练过程

如下图所示，ELMo 采用了一个两层双向 LSTM（长短期记忆网络） 网络架构很好地解决了这个挑战。

![参考：https://www.researchgate.net/figure/Bi-directional-LSTM-based-ELMo-language-model_fig5_351572119](image-6.png)

两层双向 LSTM 的设计，其每个方面都扮演着重要的角色：

1. 双向LSTM：
- 双向 LSTM 包含两个方向的 LSTM 网络，一个处理正向的文本序列（从开始到结束），另一个处理反向的文本序列（从结束到开始）。这种双向结构允许模型同时捕获前向（上下文中之前的词汇）和后向（上下文中之后的词汇）的上下文信息。
- 在传统的单向模型中，每个词只根据它之前的词来进行理解。双向LSTM通过结合来自两个方向的信息，能更全面地理解每个词在其整个上下文中的含义。
2. 两层结构：
- ELMo使用了两层 LSTM 结构，这意味着它有两个叠加的 LSTM 网络层。每一层都能够从它的输入中捕获和学习不同级别的信息。
- 第一层 LSTM 捕获和处理相对基础的语言特征（如语法结构），而第二层 LSTM 则能够基于第一层的输出进一步提炼和理解更复杂的语言特征（如语义关系）。这种层叠的方法允许模型更深入地理解文本，从而生成更精准的词嵌入。

总结来说，ELMo 的双向 LSTM 设计让模型能够有效地捕获词汇的双向上下文信息，而两层结构则使得模型可以在不同层次上学习和理解语言，这两者结合使 ELMo 在许多自然语言处理任务中都表现出色。

如前面章节所述，语言模型训练的目的是找到一组模型参数，使得在这组参数下，训练集中的 token 序列（如单词、字符等）出现的联合概率最大化。 ELMo 也不例外，假设一个长度为 N 的序列$（t_1,t_2,...,t_N）$：


1. 在已知前序序列 $（t_1,t_2,...,t_N）$ 前提下 $t_k$ 出现概率为 $p(t_k|t_1,t_2,...,t_{k-1})$，则整个序列的联合概率分布为：
![Alt text](image-7.png)
2. 在已知后续序列 $（t_1,t_2,...,t_N）$ 前提下 $t_k$ 出现概率为 $p(t_k|t_{k+1},t_{k+2},...,t_N)$，则整个序列的联合概率分布为：
![Alt text](image-8.png)
3. 综合两个方向，我们的预训练目标为最大化下述前向和后向的联合似然函数（概率取对数后由相乘改为相加，计算更方便）：
![Alt text](image-9.png)
其中 $Θ_x$、$Θ_{LSTM}$、$Θ_s$ 分别是输入层、LSTM 层以及 Softmax 层待学习的参数。预训练目的即为求解使得上述联合概率分布最大时前述参数的值。

### 4.2 ELMo 预训练模型用于下游任务

ELMo 在用于下游任务时会冻结参数，也就是相关参数期间不会被修改或微调。不同于 word2vec 只是将最终一层参数输入下游任务，ELMo 在使用时会针对具体下游任务学习一个矩阵，该矩阵会将 ELMo 各层已冻结参数进行线性组合，然后输入到下游任务中。从效果上看，ELMo 通过预训练捕获的知识为下游任务提供了附加特征。

具体使用过程如下。

预训练结束后，针对每个 token $k$，ELMo 模型训练出了大小为 $2L+1$ 的表示（representation）集合，记为 $R_k$，具体为：

![Alt text](image-10.png)

其中 $x_k^{LM}$、$\overrightarrow{h_{k,j}^{LM}}$、$\overleftarrow{h_{k,j}^{LM}}$ 分别为输入层的词嵌入表示（具体实现通过字符级 CNN 得到，这里不做重点讨论，作为默认即可）、第 $j$ 层前向 LSTM 输出、第 $j$ 层后向 LSTM 输出。

在用于下游任务时，针对 token $k$ 需要基于 $R_k$ 调整生成新的 embedding，记为 $ELMo_k^{task}$，这是 ELMo 相比以往模型最大的创新点，既保留了预训练过程的知识，又针对具体下游任务学到了新的组合方式。 的学习过程如下：

![Alt text](image-11.png)

上述公式各参数含义如下：
- $\overrightarrow{h_{k,j}^{LM}}$：这是对于第 $k$ 个 token，经过特定任务调整的 ELMo 表示。
- $E$：这是一个函数，它结合了不同层次的表示和任务相关的参数来生成最终的任务特定嵌入。
- $R_k$：这是第 k 个token的所有层次表示的集合，包括词嵌入和所有LSTM层的隐藏状态。
- $\theta^{task}$：这是特定任务的参数集合，用于调整不同层次表示的权重。
- $\gamma^{task}$ ：这是一个缩放参数，用于调整整个嵌入向量的规模。
- $\sum_{j=0}^{L}s_j^{task}h_{k,j}^{LM}$：这是一个加权和，其中 $s_j^{task}$ 是第 $j$ 层的 softmax 归一化权重，它乘以 $h_{k,j}^{LM}$，即第 $k$ 个 token 在第  $j$ 层的表示。
- $h_{k,j}^{LM}$：是 $R_k$ 中的一个元素，它表示第 $k$ 个 token 在第 $j$ 层的隐藏状态，这包括了从字符级 CNN 得到的原始词嵌入（当 $j = 0$）以及所有 LSTM 层的输出。

上述过程图形化表示如下：

![参考：https://naturale0.github.io/2021/02/24/Understanding-ELMo](image-12.png)

针对输入的每个 token，都会为具体下游任务学习生成对应的 embedding。

以上即为 ELMo 相关预训练和使用的重点内容。论文显示，采用 ELMo 后的每个 NLP 任务效果提升在 $6 - 20%$ 之间，可以说相当显著。

ELMo 解决了一词多义问题，那它有什么不足呢？相比后来的 GPT 和 Bert，LSTM 的特征抽取能力还是弱了很多。

## 5 GPT：超强特征抽取器上场

GPT 是 “Generative Pre-Training” 的简称，从名字看其含义是指的生成式的预训练。GPT 也采用两阶段过程，第一个阶段是利用语言模型进行预训练，第二阶段通过 Fine-tuning 的模式解决下游任务。

### 5.1 GPT 预训练过程

GPT采用的 Transformer 架构，但仅包含 Decoder 部分。

下图展示了 GPT 基于 Transformer 模型的神经网络架构，用于处理不同类型的文本理解任务。图中的上半部分是一个高级的分类器，它包括多个任务特定的结构。下半部分是Transformer模型的标准架构，显示了一个12层的堆叠结构，每一层由三个主要部分组成：自注意力机制、前馈网络和层归一化。

![Alt text](image-13.png)

Transformer 核心架构:
- 文本和位置嵌入（Text & Position Embed）: 输入文本被转换为嵌入向量，同时加上位置信息，以保留单词的顺序。
- Masked Multi Self Attention: 自注意力机制可以使模型在处理每个单词时考虑到序列中的所有单词，"Masked"意味着在训练过程中防止未来信息的泄露。
- Layer Norm: 层归一化有助于稳定训练过程，加快收敛速度，并改善泛化能力。
- Feed Forward: 前馈网络在自注意力后对特征进行进一步的处理。

以下是 GPT 相对于ELMo的一些潜在优势：

1. Transformer 架构:
- GPT使用的是 Transformer 架构，这种架构利用了自注意力机制，允许模型更有效地捕捉长距离依赖关系。这与ELMo使用的基于LSTM的双向模型相比，能够更灵活地处理文本中的各种结构模式。
- GPT 在每个 Transformer 层中计算自注意力，这允许它在多个层级捕捉细粒度的特征。相比之下，ELMo通过分层的LSTM来抽取特征，可能不会捕捉到 Transformer 能够捕捉到的某些复杂模式。
2. 端到端的训练:
- GPT 可以通过端到端的训练进行微调，适应特定的下游任务。而ELMo生成的词嵌入通常在特定任务上需要额外的模型层进行进一步的训练。

总体来说，GPT 的优势在于其 Transformer 架构和生成能力，这使得它在处理和理解语言时更加灵活和强大。

### 5.2 GPT 预训练模型用于下游任务

不同于 ELMo 在应用于下游任务时会冻结各层参数，GPT在用于下游任务时会连同下游任务一起进行全参数微调。

在针对下游具体任务的微调过程中包含语言建模作为辅助目标（即再次将 GPT 预训练目标纳入微调）有两个优点：

1. 改善了监督模型的泛化能力：
- 在训练有监督的模型时，主要目标是最小化在特定任务（例如分类、情感分析等）上的损失。当同时包含语言建模作为训练目标时，模型不仅要学习如何在这个特定任务上表现良好，而且还要学习生成或预测文本。这种类型的多任务学习可以促使模型学习到更通用的特征表示，从而在面对未见过的数据时，提高其泛化性能。
2. 加速了收敛：
- 收敛指的是模型在训练过程中快速达到最低损失的能力。通过在微调中包含语言建模任务，模型可以更快地调整其参数以适应特定的下游任务。这可能是因为语言建模作为辅助任务提供了额外的信号和约束，帮助模型更快地找到损失函数的最优点。

这也意味着在微调过程中，模型的损失函数将包括两部分：一部分针对下游任务（如分类的损失），另一部分是语言模型的损失（如基于预测下一个单词的交叉熵损失）。

![Alt text](image-14.png)

在GPT论文中，上述公式表示的是总损失函数 $L_3(C)$，它是由两部分组成的：下游任务的损失函数 $L_2(C)$ 和语言模型的损失函数 $L_1(C)$ 的加权和。这个组合损失函数被用来在微调 GPT 模型时同时优化两个目标：

• $L_2(C)$ 是针对特定下游任务的损失函数，比如分类、回答问题或其他类型的任务。
• $L_1(C)$ 通常是语言模型的损失，用于最大化序列的对数似然，通常是通过预测下一个单词的方式来实现的。  

其中 $\lambda$ 是一个超参数，它权衡了语言模型损失在总损失中的比重。这可以帮助模型在保持语言生成能力的同时，对下游任务进行专门的微调。在这种设置中， $\lambda$ 的值决定了在微调过程中，保留语言模型能力与提高下游任务性能之间的平衡。如果 $\lambda$ 较大，模型将更强调语言模型能力；如果 $\lambda$ 较小，模型将更专注于下游任务。

这样做的一个潜在优点是，它可以防止在微调阶段对特定任务过度优化，从而导致模型失去其原有的语言理解能力——一种被称为“灾难性遗忘”的现象。通过继续对语言模型的目标进行优化，模型能够在保持原有语言能力的同时，更好地适应新任务。

## 6 BERT：双向语言模型崛起

在 BERT 之前，语言模型不管是基于 feature 的模型（如 ELMo）还是基于 fine-tuning 的模型（如 GPT）本质上都是单向的，而 BERT 是一个真正的双向语言模型：

• GPT使用的是Transformer模型的解码器（Decoder）部分，而不是编码器。解码器在原始的Transformer模型中用于生成文本，它在生成每个新词时只能依赖于之前的词。
• ELMo 虽然表面看是一个双向语言模型，但其预训练过程不管是从左到右还是从右到左都是分开独立进行的，只是在最后进行了一个浅层的（shallow）的拼接。

### 6.1 BERT 预训练过程

BERT 预训练与微调架构如下图所示，它将预训练和微调的架构统一了起来。

![《BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding》](image-15.png)

客观地讲， BERT 在网络架构上没有创新，它与 GPT 类似都是沿用了 Transformer，它的效果拔群源自其在预训练期间采用的两个无监督任务：MLM（masked language model，掩码语言模型）与 NSP（next sentence prediction，下个句子预测）。前者用于挖掘句子内单词间的关系，后者用于挖掘句子间的关系。

#### 6.1.1 MLM 任务

标准的条件语言模型只能通过从左到右或者从右到左的方式来训练，否则每个单词会间接看到自己，这让目标训练变得像作弊。

为了解决这个问题，BERT 采用了 MLM 来解决，具体就是训练时随机遮蔽 15% 的单词（用 MASK 标记替换），训练目标为猜测这个被遮蔽的单词到底是什么。但是微调过程不会出现 MASK 标记，所以这种遮蔽导致了预训练和微调的不匹配。为了解决这个问题，BERT 针对每个被随机选中的词采用下述三种方式之一进行处理：

• （1）80% 几率被遮蔽（模拟缺失）；
• （2）10% 几率被替换为其它无关词（模拟噪音）；
• （3）10% 几率保留原词不变。

#### 6.1.2 NSP 任务

许多重要的的下游任务，比如 QA（question answering，问答） 和 NLI（natural language inference，自然语言推理），都需要理解两个句子之间的关系，而这无法通过语言模型直接捕获。为了解决这个问题，BERT 在预训练阶段加入了二值化 NSP 任务：针对我们选择的一对句子，句子 A 和 句子 B：

• 句子 B 50% 几率是句子 A 的真正后缀，标记为 IsNext。
• 句子 B 50% 几率是随机选择的并不跟随句子 A，标记为 NotNext。

然后 BERT 针对这两个标记分类进行预训练，从而学习到句子 A 和句子 B 之间的关系。

### 6.2 BERT 预训练模型用于下游任务

在传统语言模型中，处理文本对通常分为两个阶段：先独立处理每个句子，然后在某种形式上合并它们的信息。而 BERT 通过自注意力机制在一个统一框架下处理合并后的句子对。这意味着模型不仅能够在每个句子内部进行注意力分配，还能跨越两个句子进行注意力分配，即实现句子间的双向交叉注意力。这种机制使得模型能够更深入地理解两个句子之间的关系。这种统一框架使得 BERT 很容易和下游任务结合进行端到端全参数微调。

![Alt text](image-16.png)

在输入阶段，预训练中的句子A和句子B类似于：(1) 在释义任务中的句子对，(2) 在蕴含关系判断任务中的假设-前提对，(3) 在问答任务中的问题-段落对，以及(4) 在文本分类或序列标记任务中的退化的文本-空对（即只有一个文本输入而没有第二个输入）。在输出阶段，标记的表示被送入一个输出层来处理标记级别的任务，如序列标记或问答；而[CLS]标记的表示则被送入另一个输出层来处理分类任务，如蕴含关系判断或情感分析。

在 BERT 中，根据任务的不同，模型的输出被用于不同的目的。具体来说，有两种主要的输出类型：
1. Token级别的任务：对于需要在单词或标记（token）级别上进行预测的任务（如序列标记或问答系统），模型的输出是每个标记的表示（representation）。这意味着模型对输入文本中的每个单词或标记进行了编码，并产生了一个与之对应的向量表示。这些表示随后被输入到一个输出层，用于执行特定的任务。例如，在序列标记任务中，如命名实体识别，每个单词的输出表示用于判断它是否是一个实体，并确定实体的类型。
2. 分类任务：对于需要进行整体判断或分类的任务（如文本蕴含判断或情感分析），模型使用特殊的[CLS]标记的表示。在BERT中，[CLS]是每个输入序列前加的一个特殊标记，其表示被模型设计为捕捉整个序列的综合信息。这个[CLS]标记的输出表示随后被输入到另一个输出层，用于执行分类任务。例如，在情感分析中，[CLS]的表示可以用来判断整个句子或文档的情感倾向。

总结来说，BERT模型根据任务的需求，利用不同的输出表示进行处理：对于标记级任务，使用每个标记的表示；对于分类任务，则使用[CLS]标记的表示。这种设计使得BERT能够灵活地应用于多种不同的自然语言处理任务。

## 7 总结

本文描述了自然语言表示从 N-gram 到 BERT 的演进过程。其中对神经网络对自然语言的建模进行了深入描述，从 Bengio 到 word2vec 再到 Bert，每一步的演进都对技术的进步提供了巨大的推动作用。


--end--