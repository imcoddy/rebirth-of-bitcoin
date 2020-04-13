# OP_RETURN

比特币交易的全球总帐本，是公开的，匿名的，不可篡改的。账本记录了每笔交易的具体内容，以及他们被写入账本的确定的时间点（时间戳）。

如果能将现实世界的数据埋进交易，一同写到账本里，事情就变得有些微妙了，比特币系统的潜在应用将不只局限于支付，多了很多 “可玩性”。

## 一些背景

可以把一份文件的电子指纹（哈希）放到账本中，配合时间戳，建立某个确定时间点后的文件存在性证明，以此声明版权。

> to record a digital fingerprint of a file in such a way that anyone could establish proof-of-existence of that file on a specific date by reference to that transaction.

也可以直接把文件内容放到帐本中，比如合同和遗嘱。账本数据不可篡改，以此证明文件内容未经改动，是当时意愿的表达。

Coinbase 交易不需要输入，可以在腾出空间的解锁脚本中放入自定义的数据。

中本聪（Satoshi Nakamoto）在 [创世区块中留言](https://btc.com/4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b)，表达对现实金融系统的不满。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/pOg1Fft.png)

有的人在帐本里放上了 [爱情宣言](https://btc.com/d0ec21e1d73d06be76c2b5b1e5ec486085bda8264229046c11b95f66f2eded83)。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/GaO12eY.png)

早些时候的人们脑洞大开，还尝试了各种其他粗暴的方法。P2PKH 的锁定脚本里有 20 字节的公钥哈希值，有的人甚至直接将数据放到这 20 字节里。这些 “伪支付” 产生的 UTXO，因为不存在一个真实的私钥与 “伪公钥哈希” 对应，所以它们永远无法被消费，会一直存在于 UTXO 集中，导致 UTXO 数据库的大小不断 “膨胀”。

[这篇文章](https://bitkan.com/zh/ksite/articles/3821) 详细记录了各种 “伪支付” 的脑洞，[老刘 Edward](https://weibo.com/u/6387397321) 出品，良心之作，墙裂推荐你读一读。

利用比特币账本存储支付无关的数据，一直都充满争议。一方面这种使用模型充满前景，扩展了比特币系统的应用领域，但又因为没有一个统一的合适的方式，也带来了一些不好的影响。

从 Bitcoin Core 0.9.0 版本开始，通过操作码 `OP_RETURN` 最终实现了妥协。

# OP_RETURN

如果某个交易输出，其锁定脚本以 `OP_RETURN` 操作码开头：

```
OP_RETURN [数据]
```

我们称这是一个 ** 数据记录输出 **（Data Recording Output），这笔交易也被称为 `OP_RETURN` 交易，或 Null Data 交易。

对 “[数据]” 部分的长度限制，“不同版本的比特币” 也不尽相同。目前，Bitcoin（BTC）是 80 字节，而 Bitcoin Cash（BCH）是 220 字节。

Bitcoin SV（BSV）一般也限制到 220 字节，但根据 [Shadders](https://twitter.com/shadders333) 的文章 [The unfuckening of OP_RETURN](https://www.yours.org/content/the-unfuckening-of-op_return-b10d2c4b52da)，这个限制可以由 [矿池](https://aaron67.cc/2019/01/11/bitcoin-mining-consensus/#% E7%9F% BF% E6% B1% A0) 随意改动。

> The current limit on OP_RETURN data is actually a soft limit that miners are free to change. It defaults to 223 bytes but miners can raise it.

[\_unwriter](https://twitter.com/_unwriter) 在 [这笔交易](https://whatsonchain.com/tx/ef21e71d00b9fce174222e679640b09e29ac8a55f321c93e64b16cc3109959f8) 的 `OP_RETURN` 里，带上了爱丽丝梦游仙境的内容。

`OP_RETURN` 输出 ** 会 ** 随交易一同被写到账本中，但 ** 不会 ** 被当成 UTXO ，不会带来 UTXO 集的膨胀，所以其金额通常为 `0`。

任何非零金额的 `OP_RETURN` 输出，都不可消费，所以 `OP_RETURN` 还可以用来销毁（燃烧）比特币。

需要注意的是，一笔标准交易，规定最多只能有一个 `OP_RETURN` 输出。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/OXLqNXK.png)

交易 `335724b8b1a399589dfc470e474a415dc04e20d7c72e03903a2edb889ee47fde`（我隐藏了输入和第一个输出）为

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/yb7DOF1.png)

你可以用 [这个工具](https://software.hixie.ch/utilities/cgi/unicode-decoder/utf8-decoder)，看看十六进制的 UTF-8 编码 `e4bda0e5a5bdefbc8ce4b896e7958ce38082` 是什么。 😋

通过 `OP_RETURN`，数据可以被优雅的埋进账本中，下面是一些有意思的尝试。

- [Omni Layer](https://www.omnilayer.org/)，基于 Bitcoin Core 的 Token 发行方案
  - [在比特币上发代币的基本原理 ——Omni 协议发代币的通俗解释](https://zhuanlan.zhihu.com/p/40062558)
- [Wormhole](https://wormhole.cash/)，基于 Bitcoin Cash 的 Token 发行方案
  - [Wormhole 虫洞项目信息汇总](https://bitkan.com/zh/ksite/articles/14190)
- [Bitcoin Files Protocol](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md)，在比特币账本中存储文件
  - [Bitcoinfile 文件存储协议 (中文版)](https://zhuanlan.zhihu.com/p/54146706)
- [Memo](https://memo.cash/protocol)，运行在 [Bitcoin Cash](https://memo.cash/) 和 [Bitcoin SV](https://sv.memo.cash/) 上的微博
  - [基于 BCH 的永不删帖的去中心化 “微博”](https://www.jianshu.com/p/a24c1a8fe25e)

## OP_RETURN Unlimited

#TODO 增加 OP_RETURN 提高上限的背景、影响及意义。增加 OP_RETURN 将回归的说明及相关的对应。

如果你认真看过 OP_RETURN 的代码，就会发现这个操作符后面的语句是不会被执行的。这就意味着，这后面的内容是由矿工来决定的。

BSV 于去年提高了 OP_RETURN 的上限至 100k 后，涌现出了很多新的玩法。

## 链上出现非法内容怎么办？

比特币(BSV)是一个稳定的基础协议，提供了一个不可篡改的全球数据处理协议和现金网络，其底层脚本是图灵完备的，因而其上层可以产生无穷应用。既然是无穷应用，自然无法根绝黄毒赌及其他非法内容，正如 internet 和人类社会都不可能根绝这些内容。事实上，人类社会就是与这些非法内容共存、共斗争而不断进步的，关于如何处理网络上的非法内容，世界各国早已形成一套完整的法律规范，完全适用于区块链。所以，BSV 网络上如果出现非法内容并不是什么大惊小怪的事。

关键在于，BSV 是在现行法律框架内运行的，目前大概也只有 BSV 公开宣称拥抱法律。BSV 处理不了的，一定不要忘了，还有法律。代码是代码，法律是法律，任何行为，都有行为人承担责任。而且，BSV 为法律执行、尤其是追究、锁定责任人提供了不可篡改的证据线索，更有利于打击犯罪。同时，比特币的诚实系统本质，也有利于系统本身的自我净化，自我驱逐犯罪。

还有BSV的UTXO本质，主链只存储原始代码，不提供执行与解析功能，一方面主链可无限扩容，容纳无限广阔的诚实活动，另一方面也从技术上保证了非法活动可以通过独立的客户端或开发端系统直接进行屏蔽的能力。这一点，其他链，如ETH、EOS等等均无法做到，因为它们均在链上直接执行、解析代码。也正因为这一本质区分，导致BSV之外的其他链根本做不大,只能在匿名的蜗壳里闪腾腾挪，几乎只能游走于犯罪或犯罪边缘。
