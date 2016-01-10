title: "CRF Usage In Pos Tagging"
date: 2016-01-10 10:30:28
tags: [CRF, POS]
---
这篇文章中主要阐述条件随机场模型（CRF）在词性标注(POS)中的应用。其中会对比着隐马儿科夫模型（HMM）及普通的分类方法，此处不做复杂数学理论推导，只是简要概述模型的思想以及在词性标注中的应用。
<!--more-->

其实不管是HMM还是CRF，其都是应用与序列标注的问题，已经不是经典的分类算法了。对于序列标注问题，就是要考虑上下文关系，而随机过程中的马儿科夫链条就可以满足这种要求。词性标注就是一个典型的需要考虑上下文关系的序列标注问题。这里设有句子S=w1w2..wn，其中w是组成句子的单词。同时设有序列L=l1l2...ln，其中l是对应单词w的词性，且l取值于有限的词性集合，如名词，动词，介词，形容词etc...  目的就是通过大量的训练，得到模型，然后以后遇到句子S，就可以标记出单词对应的词性。  

经常接触数据处理，你可能知道这些方法分为两种，一种是生成模型（Generative model），另一种是判别模型（Discriminative model）。而HMM是通过词性到单词的生成概率来进行建模的，所以为生成模型。但是生成模型的HMM有个缺点，即从词性到单词的生成概率并没有考虑到单词本身的一些特征属性，例如以-ly结尾的单词，长度超过5的，以er结尾的单词等等。所以说，HMM是忽略了这些单词本身的属性的。但是我们知道这些属性对于词性标注这个例子又是很有帮助的，知道了一个单词以-ly结尾，很可能这个单词是一个形容词。要考虑单词本身的这些特征，而又要考虑单词与词性之间的关联关系，就需要以单词组成的句子这边为条件，即概率P(L|S)，而不是HMM里面的生成概率P(S|L).所以CRF不是生成模型，而是判别模型，即通过单词本身的属性来判断标注的概率。这就是CRF与HMM的区别，也是CRF较为先进的地方。  

其实，前面讲述了HMM到CRF的问题，下来具体陈述CRF在POS中的问题。从概率角度讲，我们就是想求出一个关于词性序列和单词序列的联合概率分布，即P(L, S)，同过概率变化可知等价于P(S)P(L|S)，其中P(S)可以很简单的从训练数据集合中统计得到。下面主要说P(L|S)的计算问题。  

P(L|S)其实是从单词的各个特征计算出来的。假设这里我们对单词选取2个特征，特征的具体定义参考下面f的定义，对于每个特征，有一个对应的权值d，其实这里就像极大熵模型中的特征选取一样。特征函数如下：

f1(S,i,li,li−1)=1 if li= ADVERB and the ith word ends in “-ly”; 0 otherwise.
* If the weight λ1λ1 associated with this feature is large and positive, then this feature is essentially saying that we prefer labelings where words ending in -ly get labeled as ADVERB.

f2(S,i,li,li−1)=1 if i=1, li= VERB, and the sentence ends in a question mark; 0 otherwise.
* Again, if the weight λ2λ2 associated with this feature is large and positive, then labelings that assign VERB to the first word in a question (e.g., “Is this a sentence beginning with a verb?”) are preferred.

有了以上的特征函数还有对应的权值，对应一个句子和其标注，我们就可以计算出一个总的权值，公式如下：  
![crf usage pos 1](/img/crf-usage-pos-1.png)

上面计算出来的可能是一个正的值，可能很大，这里只要我们使用统计学里面的任何一种归一华方法处理即可将值限定在0到1以内。如下公式：
![crf usage pos 2](/img/crf-usage-pos-2.png)

这样我们就可以通过训练数据集计算出P(L|S)，可是上面是直接使用特征对应的权值，一开始我们并不知道权值应该是多少，不过可以先随机初始化，然后使用梯度下降法通过训练数据集找出最合适的权值。  

以上就是CRF在词性标注中的主要问题。然后预测，可以使用维特比算法利用CRF马儿科夫随机场的局部独立性快速找出最优的标记序列。就这样了，我先去开会了。

[Introduction to Conditional Random Fields](http://blog.echen.me/2012/01/03/introduction-to-conditional-random-fields/)
