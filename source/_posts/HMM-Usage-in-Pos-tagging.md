title: "隐马儿科夫模型在词性标注中的应用"
date: 2015-12-24 15:50:12
tags: [HMM, POS]
---
这篇文章主要阐述隐马儿科夫模型在词性标注中的应用。词性标注POS--Part Of Speech Tagging，通俗讲就是给定一个句子S=W1W2W3...Wn，获取其对应的词性的序列T=T1T2T3...Tn的过程。其中词性是名词,动词之类的。
<!--more-->

对于词性标注这个在语音处理方面的问题，其解决方法是逐渐演变的，从最开始的人工，字典，频率，到现在基于上下文的HMM和CRF方法。  

最开始，给你一个包含许多单词的句子，要标注出各个单词对应的词性，我们可以人工标注。但是人工的问题在于不能自动化，不能批量化。所以慢慢人们就是用字典法进行标注，这就好比使用字典法进行分词一样的。字典法存在的问题在于其对多义性的处理不妥当，虽然可能有后来的启发式算法之类的，但是还不是那么Perfect。  

既然字典法也不是很完美，那么人们又想出了频率法，即当看到一个句子的各个单词的时候，我们可以根据经验，使用单词对应的频率最大的那个词性作为当前单词的词性。这样就可以选出概率最大的词性序列作为当前的单词序列的词性。那么问题来了，对于那些未登陆单词，是没有之前的频率经验的，所以就不能标注了。且频率法总是选择单个单词的大频率是明显不符合现实的。  

若是在词性标注过程中，对于遇到的单词，能考虑上下文，那么就应该更准确。例如若当前单词的上一个单词是动词，那么当前单词很可能是名词哦，例如吃饭。所以当考虑了词性序列的上下文关系后，那么标注出来的序列应该是更加准确的。这里只考虑当前词性与前一个单词的词性关系，所以这是一阶马儿科夫性质，就是说词性序列构成了一个马儿科夫链条。而每个词性都对应这一个单词。如下图所示：
![HMM](http://csrgxtu.github.io/img/HMM_and_CRF-300x98.png)

因为一般算法直接看到是单词序列，即可观测序列，而词性序列不是直接观测到的，而是计算出来的，所以在这里我们称之为隐藏的。所以可以描述上述的马儿科夫词性标注过程为隐马儿科夫模型。  

其实隐马儿科夫模型并不是数学家马儿科夫提出来的，而是由美国的做语音处理的一群科学家逐渐应用并提出来的。

前面只是讲了事情是如何一步一步发展到隐马儿科夫模型的，下来将简单阐述HMM在POS中的一些具体问题。  

从概率论的角度看POS问题，就是就是求一个词性序列T=T1T2T3...Tn的分布，即_P(T) = P(T1, T2, T3, ..., Tn)_的问题，但是前提是在观测序列的条件下，所以又可以认为是求_P(T1, T2...Tn|W1, W2...Wn)_，即P(C|W)。又因为在概率论里面，基本上所有的计算都可以使用求和，连乘，贝叶斯公式灵活转换。所以P(C|W)就可以P(C,W)/P(W),当然也可以P(W|C)P(C)/P(W).  

其中，P(W)是单词的频率分布，这个可以从大量的语料库文本中统计出来，P(C)是词性序列之间的转移概率，也是可以从语料库中统计出来，P(W|C)是在词性的条件下，单词的概率，即生成概率，也是可以从语料库中统计出来的。  

以上基本上就知道了一个HMM模型的主要要素，所以现有一个词性序列T=T1,T2...Tn，就可以通过上述公式计算出来这个词性序列的概率。但是有那么多的序列都需要去计算，然后选择概率最大的那一个序列，这样也是有很大的计算量，感觉就是穷举法。所以有一个牛逼的人叫Veterbi使用Veterbi算法即最短路径算法可以在有限的时间内找到最优的词性序列。  

这里我没有给出HMM的定义，只是在闲聊中做了一个简单的阐述。就这样了，我准备去爬山了。
