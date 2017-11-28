---
layout: post
title: learn the structure of SPN --notes
date: 2017-11-28
categories: SPN学习，深度学习
tags: [深度学习，SPN学习]
description: 论文学习笔记：
---

[论文地址](http://proceedings.mlr.press/v28/gens13.pdf)

# Learning the Structure of Sum-Product Networks————笔记

在已知一个SPN的结构时，它的权重可以通过生成学习或者判别学习得到。Poon在2011年提出的生成模型中，每个实例都被看作是一个spn实例，因而得到的最大后验概率子节点的权重是逐渐递增的。但是该文章使用了预先设计的spn结构，从而丧失了灵活性。对于判别学习，Gens提出的反向传播梯度下降算法，用于处理图片分类问题，但是依然不能权衡学习的消耗和得到的模型的灵活性。

本文提出了第一个不牺牲spn表达能力的学习算法。先提出一个基于简化SPN的定义，抛弃了原先使用的指示函数和网络多项式，对于连续变量而言生成过程更加简单。

SPN的定义：

1. 一个$O(1)$时间内能计算得到配分函数和边缘分布的单变量分布是一个SPN。

2. 有不相关域的多个SPN的积是SPN。

3. 多个具有相同域的SPN的带权和是SPN，权重必须是正值。

4. 除此之外的任何模型都不符合SPN。

本文 使用的方法：

- 实例成簇：EMS算法。

- 变量分离：朴素贝叶斯混合模型。

本算法的输入是一组向量-值形式的变量，以矩阵`实例×变量`的形式。如果这个向量是单位长度，那么该算法返回对应的单元素的分布，同时返回参数的最大后验估计。例如：对于离散变量，返回的分布可能是带有狄利克雷先验的多项式分布。对于连续变量，返回的可能是一个带有正态wishart先验的正态分布。如果向量长度大于1，则该算法会在向量的行子集或者列子集上递归地执行。如果能将矩阵中的变量分离成互相分离的变量子集，那么它会在变量子集上执行算法，并且返回所有子集上算法返回值的积。否则，它会将输入实例簇化分成多个相似的子集，并在子集上执行算法，返回各个子集上算法返回值的加权和，每个子集上的权重是该子集中实例占对应簇的实例的比例，即计数比例，权重也可以通过狄利克雷先验进行平滑。

- 递归执行：

![sum-product-分离](http://willis-hu.github.io/my_pic/sum-product-分离.png)

如果没有发现变量子集之间的关联关系，则算法返回一个*全因式分解*的分布。另一方面，若算法递归时没有找到相互独立的变量子集，直到$|T=1|$，即所有变量之间都是相互独立的，则算法返回分布的核密度估计。第一个条件相当于变量子集中所有变量都是相互关联的，第二个条件相当于算法中所有变量都是相互独立的。更一般地，若$|T|>>|V|$，则算法会递归到实例子集上执行，若$|T|<<|V|$，则算法会递归到变量子集上执行。该算法可以在不同的实例子集上采取不同的变量分离策略，这可以保证模型中有较少的条件独立性。

- 变量无关联关系

![no dependency find](http://willis-hu.github.io/my_pic/no-dependency.png)

- 变量全部相互独立

![never find independence](http://willis-hu.github.io/my_pic/never-independence.png)

LearnSPN应该看作一个算法方案而不是一个单一的算法，该算法中可以包含多个分离变量或者实例的方法。此外，实例可以通过EM算法进行聚类，本文使用的便是该方法。在每个分离步骤，我们假定一个朴素贝叶斯混合模型，在该模型中所有变量在聚类$P(V)$条件下相互独立，其中$P(V) = \sum _iP(C_i)\prod _jP(X_j|C_j))$，其中，$C_i$是第i个聚类，$X_j$是第j个变量。（*注：该公式表示该簇中所有变量都相互独立，联合概率法可以通过直接相乘计算*）。对于`soft EM算法，实例可以被部分地分配到聚类中，T需要为每个实例赋予一个权重，并且实例将会被归类到它具有非零权重的聚类中。然而，这种方法并没有hard EMS算法有效。hard EM中，每个实例会被分配到它具有最大概率的聚类中，在这里我们使用hard EM 。我们使用online EM算法时，每个实例会被分配进它具有最大概率的聚类中，聚类数量会自动生成，通过给予聚类数量一个指数分布的先验来避免过拟合现象。

在算法中的每个实例分离步骤中，算法都局部最大化了子SPN图的后验概率。混合模型的后验概率也是算法返回值的下届，因为**当在簇内独立变量子集合上调用算法时，整个簇的后验概率会始终增加**。（注：*这段话主要表明实例分离步骤最大化了子spn图的返回值*）

一个实例聚类方法为：根据实例中某个关键向量或者某些关键向量集合进行聚类。可以根据**互信息**选择最佳关键向量。这使得算法有了学习上下文无关图模型的某些特点。

变量分离同样可以采取多种方式，互信息可以看作一种**次模函数**，Queyranne算法可以被用来在立方时间内找到将变量分为两个包含最小经验互信息的子集。然而，我们发现这种方法运行过于缓慢，作为改进，我们仅考虑具有相互关联关系的两个变量。这种情况下，我们在每两个变量上进行独立性检测，每一对关联变量之间都用边相连，以此成图，进而在图的连接部分里递归进行分离。（*注：若变量有独立子集则会形成多个独立子团，在子团上递归*）。若图中只有一个连接部分，则变量之间都是关联的，变量分离失败，算法会跳转到实例分离部分。

假设我们定义**独立准则**,它表示两个变量集合$V_1$和$V_2$是独立的，当且仅当$V_1$和$V_2$中所有子集都是独立的。使用上述准则时，将子SPN分解成连接部分则不会导致似然损失。假设粒度序列表示学习算法的每个步骤中形成聚类的数量，并且算法中实例分离采用   soft EM，单元素分布使用最大似然估计，那么该算法返回一个局部最优SPN。这可以用以下命题表示：

- 给定一个粒度序列，通过独立准则进行变量分离，EM算法进行实例聚类，学习该SPN将返回一个局部最优SPN。