﻿---
 NLP预训练-语言模型的条件概率计算
---



这两年NLP中最火的就是几个预训练模型了，要真正理解这些强大的预训练的模型，需要从最基础的语言模型开始。


**(1)何为统计语言模型？**

一个语言模型通常构建为一句话的概率分布p(W)，这里的p(W)实际上反映的是W作为一个句子出现的概率。 

说成大白话，语言模型就是求一句话出现的概率。


**(2)如何构建统计语言模型**

对于一个由T个词按顺序构成的句子，p(W)实际上求解的是字符串的联合概率，利用贝叶斯公式，链式分解如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190722100844549.png)

从上面可以看到，一个统计语言模型可以表示成，给定前面的的词，求后面一个词出现的条件概率。


我们在求p(W)时实际上就已经建立了一个模型，这里的诸多条件概率就是模型的参数。如果能够通过语料，将这些参数已学习到，就能够计算出一个句子出现概率。


**(3)如何计算上面的概率？**

先提出一个问题：
有语料集D，如何计算一个句子W(w_1,w_2,...w_T)出现的概率？


根据统计语言模型，有如下的公式：  

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019072210091480.png)
那么要计算句子W(w1,w2,...wT)出现的概率，需要计算所有的条件概率P(w_t|w_1，w_2, ..., w_t-1)   1<t<T


容易想到：

       P(w_t|w_1，w_2, ..., w_t-1) = count(w_1，w_2, ..., w_t)/count(w_1，w_2, ..., w_t-1)


因此，计算句子W(w1,w2,...wT)出现的概率，就只需要统计句子内不同长度子序列在语料集内出现的次数。


当语料集非常大时，这是一个非常耗时的工作。因此，就有人提出了一个偷懒的办法。在计算 P(w_t|w_1，w_2, ..., w_t-1) 时，不在考虑w_t前面所有的字，而是仅考虑其前面n个字，因此这种模型成为n-gram。


这样，条件概率的计算公式就变成：

    P(w_t|w_1，w_2, ..., w_t-1) =  count(w_t-n，w_t-n+1, ..., w_t)/count(w_t-n，w_t-n, ..., w_t-1)


这种办法能够极大的减小计算量，但是问题。

(1) count(w_t-n，w_t-n+1, ..., w_t) =0时，是不是就认为P(w_t|w_1，w_2, ..., w_t-1)=0？

(2) 假如count(w_t-n，w_t-n+1, ..., w_t)=count(w_t-n，w_t-n, ..., w_t-1),是否就能认为P(w_t|w_1，w_2, ..., w_t-1)=1？

显然是不能的，所以n-gram模型，通常需要做平滑处理。


总的来说，n-gram模型的主要工作就统计语料集中各个子串出现的概率，再加以一定的平滑处理，就可以计算句子出现的概率了。


这是早期的统计语言模型的内容了，那么有没有更好的办法呢？这就轮到神经网络“粉墨登场”了。


神经网络语言模型(NNLM)

对于语料集D，语料中任意一个词w,取其前面n-1个词，构成context(w)。构成一个二元对，(contenxt(w),w)就是一个样本。

NNLM的基本思想是，构造一个神经网络，利用语料进行训练，之后就可以利用神经网络前向传播计算条件概率了。NNLM神经网络的结构如下，非常的简单。其中，特别重要的一点是，他将输入的词进行了向量化的处理，即每个词的输入变成了一个m维的向量。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190722101028313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hCX3BsZWFzZQ==,size_16,color_FFFFFF,t_70)
样本中，context(w)作为输入，经过向量化之后，concate起来，再经过tanh函数和softmax函数，预测w_t。

