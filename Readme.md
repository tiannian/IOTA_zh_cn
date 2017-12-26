# The Tangle
Serguei Popov

October 1, 2017. Version 1.3

> 译者注：本文是对IOTA白皮书的初步翻译，有条件的读者最好选择直接阅读英文原文。有对此翻译的建议可以提出issues，参与翻译可以直接提出pull request。

> 原文位于[`https://iota.org/IOTA_Whitepaper.pdf`](https://iota.org/IOTA_Whitepaper.pdf)

翻译：

[tiannian](http://github.com/tiannian) dtiannian@aliyun.com

## Abstract
在这篇文章里我们分析了一种为物联网设计的加密货币IOTA的数学基础，这种货币的主要特点是利用有向无环图（DAG）存储交易的Tango。Tango会成为区块链下一步的未来，它将提供对于M2M小额支付的系统支持的特性。

本文的重要贡献是马尔可夫链蒙特卡洛算法族，这些算法在为刚到达的交易选择在Tango中的顶点(site)。

## 系统介绍和描述

过去6年里比特币的兴起和成功证明了区块链技术的价值。不过，这项技术也有很多的缺点，导致它无法被用作全球加密货币的通用平台。一个值得注意的缺点是交易费用的概念。在快速发展的物联网产业中，微支付的重要性将会增加，而且支付的费用也会比价值转移的数量要高，这是不合理的。此外，在区块链基础设施中去除费用并不容易，因为它们是区块创建者的动力。这就导致了在现有的加密货币技术中的系统异构性。系统中有两种不同类型的参与者，一类是发布交易的，另一类是批准交易的。这种系统的设计不可避免的会产生参与者的区别，造成冲突，使得所有对象需要花费资源来解决冲突。而解决前面提到的问题的方案是寻找与比特币和其他加密货币基础不同的解决方法。

在本文中，我们讨论了一种不包含区块链的创新方法，这种专门为物联网行业设计的加密货币被称为IOTA。本文主要关注与Tango的特征根本，并讨论当试图摆脱区块链和维护分布式账本时出现的问题。具体的IOTA协议本文没有讨论。

一般来说，基于Tango的加密货币工作方式如下：用于代替全局区块链的DAG被称为Tango。用于存储交易的账本是一张Tango图，由节点发出的交易作为图的顶点集。Tango的边集是通过如下方式产生的。当一笔新的交易到达时，它必须认可两笔之前的交易。这些认可关系由有向边表示，如图1。如果交易A和交易B之间没有有向边，但是从A到B有一条长度至少为2的定向路径，我们则说A间接认可B。被其他交易直接或间接认可的交易为创世交易（图2）。创世交易的描述如下：在Tango的开始，有一个包含所token的地址。创世交易将这些tokens发送给其他几个“创建者”地址。在这里我们要强调，所有的token都是在创世交易中被创造出来的，未来不会创造token，从这种意义来说，矿工不会从虚空中获得金钱奖励。

关于术语的简要说明：顶点在Tango图上表示的是交易，网络由节点组成;也就是说，节点是发布和验证交易的主体。

Tango的主要思想是：为了发布一笔交易，用户必须批准其他的交易，因此发出交易的用户参与网络的安全性。节点将会检查被批准的交易是否冲突，如果一个节点发现一笔交易与Tango历史冲突，节点将不会以直接或间接的方式批准冲突。

随着交易获得更多的批准，它将会被系统以更高的信任度所接受。换句话说，让系统接受双重花费的交易将会变得非常困难。请注意，我们并没有规定选择节点将批准哪些交易的规则。如果大量的节点遵循某个“参考”规则，那么对于任何固定节点，最好坚持一个相同的规则。这似乎是一个合理的假设，尤其是在IoT环境中，节点是会是预先烧录固件的专用芯片。

为了发出一笔交易，节点执行如下操作：

- 节点根据算法选择两个交易来进行批准，一般来说这两笔交易应该符合
- 节点检查两笔交易是否冲突，并且不会批准冲突
- 为了使节点发出有效的交易，节点必须解决类似于比特币区块链中的密码难题。这是通过找一个nonce来实现的，这个nonce与来自被批准的交易一些数据连接起来的哈希具有特定的形式。在比特币中，哈希必须至少具有预定数量的前导零。

值得注意的是，IOTA网络是异步的，节点不需要看到相同的一组交易。同时还应该注意到，Tango中可能包含相互冲突的交易。这意味着在Tango里的节点不必要为账本中的全部有效交易达成共识。但是在交易冲突的情况下，节点需要选择那些交易将成为孤立的交易。节点用于在两个相互冲突的交易之间作出决定的主要规则如下：一个节点运行tips选择算法（见章节4.1）多次，并找到这两笔交易中哪一笔更有可能被已选择的交易简介的批准。例如：如果一笔交易在100次tips选择算法运行中被选择了97次，我们则说它被97%的信任确认。

让我们再讨论如下问题：是什么促使节点传播交易？每个节点都会计算一些统计数据，其中之一是从邻居接受了多少新的交易。如果一个特别的节点“太懒”，那么他将会被他的邻居剔除。因此，即便一个节点不发送交易，并且没有直接的动机去分享新的交易去批准自己的交易，它仍然有参与的动机。

在第2节引入一些符号之后，我们讨论用于选择两个交易去批准的算法，度量交易 overall approval 的规则（章节3,尤其是章节3.1），和可能的攻击场景（章节4）。同时，在读者惧怕公式符号的情况下，他们可以直接跳转到在每个部分末尾的“结论”。

应该注意的是在加密货币体系中使用DAG的想法已经存在有一定的时间了，参见[3,6,7,9,12]。特别在[7]中，介绍了GHOST协议，该协议提出了一种修改比特比协议的方法，即将主账本作为树而不是区块链。结果表明，这种修改减少了确认时间，提高了网络的整体安全性。在[9]中，作者考虑一个基于DAG的加密货币模型。他们的模型与我们的模型不同，原因如下：他们的DAG的定点是块而不是每个交易;他们系统中的矿工竞争交易费用;而新的toke可能由创建块的矿工创建。另外，请注意，尽管[6]中提出了一种与我们的方法类似的解决方案，然而它并没有讨论任何特定的tip批准策略。在本文的第一版出版后，出现了一些旨在创建基于DAG的分布式账本的其他方向，例如[8]。我们也参考了另外的方法[2,10]，旨在通过建立点对点支付渠道使比特币微支付成为可能。

## 权重和更多
在这一部分，我们定义了交易的权重以及一些相关的概念。交易的权重与发出交易的节点的工作量成正比。在IOTA当前的实现中，权重只可能取值3^n，其中n是一个正整数，取值是可接受的非空正整数值。事实上如何在实现中获得权重是无关紧要的，重要是的是每个交易上都需要被附着一个正整数。总体来说这个想法的重点是权重较大的交易比权重较小的交易更加“重要”。为了防止垃圾攻击以及其他类型的攻击，我们假设没有任何一个实体可以短时间内产生大量“可接受”权重的交易。

我们需要关注的另一个概念是交易的积累权重：它被定义为特定交易的自身权重加上直接或间接批准该交易的所有交易权重之和。图1展示了积累权重的计算方法。这些方框代表交易行为，每个方框SE角落的小数字代表交易自身的权重，加粗的数字代表积累权重。例如，交易F被交易A，B，C，E直接或间接的批准，F的积累权重就是9=3+1+3+1+1，这是F自身的权重和A，B，C，E的权重之和。

让我们定义“tips”作为在Tango图中未批准的交易。在图1上部的Tango中，只有A和C是tips。在图1下部的Tango中，当新的交易X到达并被A和C批准之后，X就成为了唯一的tip。所有其他的交易都权重都增加了X自身的权重3。

我们需要引入额外的两个变量来讨论批准算法，首先，对于Tango上的顶点，有两个参数：
- 高度：通向创世交易最长路径的长度
- 深度：通向tips最长的反向路径长度

例如：在图2中G的高度是1，深度是4，因为反向路径是F，D，B，A，而D的高度是3深度是2。接下来让我们介绍下分数的概念：根据定义，交易得分是本次交易批准的所有交易自身权重加上交易本身自身的权重之和。根据定义，交易得分是本次交易批准的所有交易自身权重加上交易本身自身的权重之和。在图2中，只有A和C是tips。交易A直接或间接的批准了交易B，D，F，G，所以A的分数是1+3+1+3+1=9。类似的，C的分数是1+1+1+3+1=7。

为了理解本文中提出的观点，我们可以安全的假设所有交易所拥有的自身权重均等于1。从现在起，我们坚持这个假设。在此假设下，交易X的累计权重等于1加上直接或间接批准X的交易数量，得分为1加上由X直接或间接批准的交易数量。

我们应该注意到，在本节中定义的那些之中，累计权重是（迄今为止）最重要的指标，尽管高度，深度和分数也需要进一步的讨论。

## 系统的稳定性和cutsets
设L(t)为在t时刻系统中tips的总数。人们希望随机过程L(t)趋向于稳定。更确切的来说，希望过程是常返的，正式定义参见4.4部分和6.5文档。

为了分析L(t)的稳定性，我们需要作出一些假设。一个假设是交易是由大量独立的实体发出的，那么到来的交易的过程可以用泊松点过程来建模。设\lamdba是泊松过程的速率。简单起见，我们使这个这个速度保持不变。假设所有的设备具有大致相同的计算能力，那么设h是设备为了发出交易而所需执行计算的平均时间。然后假设所有节点的行为如下：为了发布交易，节点随机选择两个提示并批准它们。应该注意到的是，对于采用这种策略对于“诚实”节点来说不是一个好策略。特别的是没有足够的保护来对抗“懒惰”或者恶意节点。另一方面，我们仍然会考虑这个模型，因为它很容易分析，并且可以提供对系统行为的深入了解，以便得到更复杂的tips选择策略。[...未完成...]

接下来我们做进一步的简化，对于任意一个节点，当它发出一个交易时，关注的不是Tango的实际状态，而是一个确切的时间单元。这意味着对于在时间t时一个交易，在时间t+h的网络上可以看到一个交易。我们还假设tips的数量在时间上大致保持不变，并且稳定在L0>0。下面我们计算L0作为\lambda和h的函数。





## 可能的攻击

### 寄生链攻击和新的tips选择算法

### 分裂攻击

## 抵抗量子计算

## 感谢

## 引用
