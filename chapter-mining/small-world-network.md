# 小世界网络

## 什么是小世界网络？

在数学、物理学和社会学中，小世界网络是一种数学上图的类型，在这种图中大部分的结点不与彼此邻接，但大部分结点可以从任一其他点经少数几步就可到达。若将一个小世界网络中的点代表一个人，而连结线代表人与人认识，则这小世界网络可以反映陌生人由彼此共同认识的人而连结的小世界现象。

![small-world-network](http://i1375.photobucket.com/albums/ag455/imcoddy/bitcoin/small-world-network_zpspsjygfkn.png)

比特币节点组成的网络是一个小世界网络，这是论文《[Exploring the Bitcoin Network](https://www.researchgate.net/publication/262562539_Exploring_the_Bitcoin_Network)》中得到的结论:

> One important result concerns the relationship of network usage and exchange rate, where a strong connection could be confirmed. Moreover, there are indicators that the Bitcoin system is a “small world” network and follows a scale-free degree distribution.

这意味着，比特币网络中节点的紧密度比许多人想象的要密得多。

## 小世界网络的意义

小世界网络的意义在于，参与的节点大部分会紧密连接，使得信息广播在整个网络中的得以高效传递。同时这些节点组成的网络具有良好的扩展性，新加入的节点可以根据自身情况迅速融入其中，退出的节点也不会影响到整个网络系统。

在谈论网络拓扑结构的时候，人们通常会认为是以下几种形式。

![Centralised-decentralised-and-distributed-networks](http://i1375.photobucket.com/albums/ag455/imcoddy/bitcoin/Centralised-decentralised-and-distributed-networks-Baran-in-Barabasi-2003_zpsxnpqulzx.png)

作为一个去中心化的网络，比特币网络很容易让人认为它是处于上图中第二和第三种状态之间。这样的网络结构也被称为 [Mesh 网络](https://zh.wikipedia.org/wiki/% E7% BD%91% E7%8A% B6% E7% BD%91% E7% BB%9C)。在这种状态下，平均每十分钟一个区块要传播至全网，需要通过若干节点的几跳（相邻节点的一次传递叫一个跳）的传递，才能使得区块从网络的一端传递至另一端。

而在实际情况中，节点会倾向于与连接性良好的其它节点保持紧密关联，以便能尽快地接收交易信息以及将自己打包的区块传递出去。这使得节点在加入比特币网络后，会尽量与更多节点直接关联，从而让整个比特币网络慢慢变成了一张近完全图（Near Completed Graph)。下面的视频，生动地显示了新节点不断加入比特币网络后整体变得更加紧密的过程。

[![Block Propagation](https://img.youtube.com/vi/D0Jqah-DrpU/0.jpg)](https://www.youtube.com/watch?v=D0Jqah-DrpU)

(视频需要翻墙 [https://www.youtube.com/watch?v=D0Jqah-DrpU](https://www.youtube.com/watch?v=D0Jqah-DrpU))

小世界网络使得在比特币网络中广播区块交易信息非常高效，平均在两跳之内即可将信息同步到大部分的节点。这也是比特币网络自然进化的结果：各个节点之间是竞争关系，因为它们需要争取下一个块的打包权；同时他们又不得不与其它节点保持良好沟通，以保证自己在挖矿过程中不处于劣势。

## 小世界网络是否让比特币变得中心化？

这是一个很多人无法理解的坎：要尽可能地与更多节点保持连接意味着需要提升节点的硬件性能。如果最后变成了都是服务器级别的节点，这不会让很多爱好者无法自己在家运行，最后只有少数几个大矿场参与的游戏？

这是一个很值得讨论的问题。

关于去中心化的更详尽讨论，我今后会另外撰文。在这里，希望读者能够先思考一下：去中心化的度应该如何衡量？在人人自建节点挖矿和只有屈指可数的节点参与之间，怎么样算足够去中心化？如何尽量保证比特币网络能够足够分散而不会因为单点故障而崩溃，同时又能尽可能保持整个网络高效运作？

中本聪在比特币初期，孤独地一个人挖了一年之久，那时的比特币网络是中心化的，因为如果中本聪关了机后整个网络将停止运作。但那也是去中心化的，因为它对任何人开放，也随着后来加入的人越来越多，一直发展至今天的模样。

因此，对于这个问题，比特币的节点组成的小世界网络是对外开放的，它不像 EOS 这样指定了候选人，任何节点都可以自由地加入其中并有机会挖到块。同时这也是竞争激烈的，通过优胜劣汰的方式，让一些过时的设备不得不出局，从而让整个网络的承载能力能逐渐提升。
