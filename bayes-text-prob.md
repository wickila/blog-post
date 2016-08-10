---
title: 朴素贝叶斯文本分类算法中的概率公式解释
date: 2016-08-10
category : 数据挖掘
published : True
tags : [数据挖掘，朴素贝叶斯，文本分类，概率]
---

最近在github上学习[朴素贝叶斯文本分类]([https://github.com/egrcc/guidetodatamining/blob/master/chapter-7.md](https://github.com/egrcc/guidetodatamining/blob/master/chapter-7.md))，在练习中遇到一个比较难理解的公式——计算某个单词在某个分类中出现的概率。公式如下：

$$
P(w_{k}|h_{i})=\frac{n_{k}+1}{n+|Vocabulary|}
$$
其中: 

- $n_{k}$为单词w在h分类中的个数。


- n为h分类中总单词个数。
- Vocabulary为样本的总词汇量（注意是总词汇量，而不是总单词个数）

<br>

这个公式，一眼看上去比较复杂，不太容易看懂。我们可以先从简单的公式入手：
$$
P(w_{k}|h_{i})=\frac{n_{k}}{n}
$$
即单词在分类中出现的概率＝单词在分类中出现的次数／分类的总单词数。这个公式就是按照普通的概率论得来的公式，我在练习中第一反应得出来的也是这个公式。

然而，在实践中，有的单词在某个分类中不会出现，从而会导致贝叶斯算法的出来的概率里面，这个分类的概率就是0 ——0概率的问题作者在[教程第六章](https://github.com/egrcc/guidetodatamining/blob/master/chapter-6.md)中有提到解决办法，即分子加mp，分母加m。我们的公式就变成了下面这样：
$$
P(w_{k}|h_{i})=\frac{n_{k}＋mp}{n+m}
$$
在我们这个算法中，m就是样本词汇量|Vocabulary|，p就是单词w在样本词汇中出现的概率，即1/|Vocabulary|，把m与p代入到上面的公式。就得出了算法中的公式。

所以，我们在算法中遇到的这个公式，不是从数学理论得出来的公式，而是在实践中经过调整后得来的公式。记住这一点之后，这个公式就变得清晰了。