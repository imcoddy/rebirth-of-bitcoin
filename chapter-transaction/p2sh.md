# P2SH

了解复杂的交易类型，能帮助你更好的理解，什么是 “可编程加密货币”。

## 付款到多重签名

如果你看过 `OP_CHECKMULTISIG` 操作码的说明，你会发现：交易允许我们将 UTXO 锁定到 $N$ 个公钥上，并设置花费条件，需要至少提供 $M$ 个签名，才能解锁资金。

这种方案，称为 **M-N 多重签名 **，其中 $N$ 是密钥的总数，$M$ 是验证所需的最少签名数。

```
M [公钥 1] [公钥 2] ... [公钥 N] N OP_CHECKMULTISIG
```

如果锁定脚本是这样的形式，我们称这是一笔 ** 付款到多重签名 **（P2MS，Pay to Multi Signature）的交易，也常用 multisig 表示。

解锁 UTXO 时，需要提供的解锁脚本，形如

```
[签名 1] [签名 2] ... [签名 M]
```

这个新特性由 [BIP-11](https://github.com/bitcoin/bips/blob/master/bip-0011.mediawiki) 提案引入。

BIP 是 ** 比特币改进建议 **（Bitcoin Improvement Proposal）的缩写。

- [BIP 是什么，跟比特币是什么关系？](https://www.zhihu.com/question/269330181)
- [比特币的神奇仪式：比特币改进协议（BIP）是如何产生的？](https://www.8btc.com/article/103292)

一个 BIP 提案有九种状态。

- Proposed（提出）
- Draft（草案）
- Active（激活）
- Final（落地）
- Replaced（被替代）
- Withdrawn（撤回）
- Deferred（推迟）
- BIP number allocated（BIP 编号被分配）
- Rejected（拒绝）

正常的，操作码 `OP_CHECKMULTISIG` 在执行时，应该

- 先弹出一个数，为 $N$，知道有多少公钥
- 接着弹出 $N$ 个公钥的值
- 再弹出一个数，为 $M$，知道有多少签名
- 接着弹出 $M$ 个签名的值
- 得到了所有的 $(N + M + 2)$ 个数据，计算脚本是否有效

但 `OP_CHECKMULTISIG` 执行时有个 bug，在弹出 $M$ 个签名之后（这时栈已经空了），会再做一次出栈操作。这将导致栈错误，脚本执行失败，交易被标记为无效。

为了避免这种情况，默认会在解锁脚本的开头加一个额外的项，可以是任何值，通常是 `0x00`。

这个额外的值，在 `OP_CHECKMULTISIG` 执行过程中并没有被使用，所以它不影响整体逻辑的正确性，它的存在只是为了避免栈错误。

所以正确的解锁脚本为

```
0 [签名 1] [签名 2] ... [签名 M]
```

这个小彩蛋，被永久的留在了系统中。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/OXLqNXK.png)

交易 `6107e9397f6f3bc881e542923fed370f8d623daa3858078827c97fa26449a127` 是一笔 multisig，有 3 个输出。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/cMym4AY.png)

第二个输出，被锁定成 `1-1` 多签，在交易 `9395df4d739596ae9871b2d2a6017beb0982c4c79e634d5752976e10f4b63847` 中消费，是这笔交易的倒数第五个输入，提供的解锁脚本为

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/BTFXbTR.png)

解码一下。

```
0  <== 这是彩蛋
30430220122b6a64e64a01f3bf490c8295a811f9f844bb59b2bd10c18ed1dffe7fd2f97d021f3a15805efbaac6b407ec663e743637b4c45b57316639b1f60c8675a3af38db[ALL]
```

容易想到 multisig 的使用场景，常见的，

- `1-N` 多签，用于多人共享资金，组内的任何人都可以直接支付
- `N-N` 多签，用于多人的共同存款，必须所有人同意才可以支付
- `2-3` 多签，用于公司财务，必须超半数以上的人同意才可以支付

一般的，把由 [BIP-11](https://github.com/bitcoin/bips/blob/master/bip-0011.mediawiki) 定义的多重签名方案，称为 old-style multisig。

这种多签方案非常直观，容易理解，但不方便使用。

对于 P2PKH，只需要知道收款人的公钥哈希，或 “公钥哈希的某种可逆编码”，利用 “钱包” 软件就可以直接生成交易，因为锁定脚本的形式是确定的。

```
OP_DUP OP_HASH160 [公钥的哈希] OP_EQUALVERIFY OP_CHECKSIG
```

对于 old-style multisig，必须要知道收款人多签的全部细节，才能确定锁定脚本。

- $M$，至少需要几个签名解锁，1 字节
- $N$，一共有多少公钥，1 字节
- 每个公钥的具体内容，至少 $33N$ 字节

如果一个公司使用多签收款，他必须提前把脚本内容发送给所有客户。同时还要求他的客户，能使用特制的 “钱包” 软件，生成包含复杂脚本的交易，完成支付。并且，这个锁定脚本的内容非常长。

大多数比特币交易都包含交易费，这是对 “维护比特币网络安全的人” 的一种激励和补偿。

你现在需要知道，** 应付交易费的多少，只与交易数据序列化后的大小有关，而与这笔交易转移价值的多少无关 **。

所以 old-style multisig 还需要交易付款方，支付更多的交易费。

## 付款到脚本哈希

复杂的支付脚本功能强大，但在使用时有诸多不便，因为付款方需要了解锁定脚本（由收款方定义）的全部细节。

为了避免这种问题，[BIP-16](https://github.com/bitcoin/bips/blob/master/bip-0016.mediawiki) 提案引入了全新的交易类型，允许我们将 UTXO 锁定到一个脚本的哈希（数据指纹）上。

如果 ** 锁定脚本中关联的是某个脚本的哈希 **，

```
OP_HASH160 [脚本的哈希] OP_EQUAL
```

我们称这笔交易，是一笔 ** 付款到脚本哈希 **（P2SH，Pay to Script Hash）的交易。

P2SH 的含义是，向 ** 与该哈希匹配的脚本 ** 支付，这个脚本被称为 ** 赎回脚本 **（Redeem Script），其内容将在之后，在消费这个 UTXO 时呈现。

> When a transaction attempting to spend the UTXO is presented later, it must contain the script that matches the hash, in addition to the unlocking script. In simple terms, P2SH means "pay to a script matching this hash, a script that will be presented later when this output is spent."

支付时需要提供的解锁脚本，形如

```
[参数 1] [参数 2] ... [参数 X] [赎回脚本的内容]
```

验证分两部分，先计算赎回脚本的哈希，看它是否跟锁定脚本中的脚本哈希一致。

```
[赎回脚本的内容] OP_HASH160 [脚本的哈希] OP_EQUAL
```

如果一致，再执行解锁脚本。

```
[参数 1] [参数 2] ... [参数 X] [赎回脚本的内容]
```

有点不太好理解。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/OXLqNXK.png)

交易 `d957651a876addc3a4e836c0f55d3e288230c9622f7062a9c1d963480768726e` 是一笔 P2SH，有 2 个输出。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/0T0L9le.png)

第一个输出，被锁定到一个脚本哈希上，值为 `8bd55244e4f86fb631e908f8cd9d9084e6744ad1`。

这个 UTXO 在交易 `536749e6a0cb146287ec1ceffe50a65c3760d794aacb40367239cb3f332c6ba5` 中消费，是这笔交易的第一个输入，提供的解锁脚本为

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/UUXT9C9.png)

解码一下。

```
0
3045022100cd2eff6b93874c822c5496a2fd660f3f0a09e8dc40e504b14a5fbd38bcfff4db02205df3ad1e6a28e762b471012ee0b1e067cb39bbe4a3494c1144b157ccef25bc71[ALL]
3045022100ea4631ed3e9ae30f4faaa17b396398b30959bd119558349e4aa40ecb75856c0e0220684be10d059339533d225cd21e75079dc771e10ebe8f0358db3ec18763e34f22[ALL]
522102429adbb84a4a0f14b31c14f4927418207bcef7f70eb97b1caed49160733bff6921026ce3c7280d473b7a9eab8fe76219687deb646c1619ad18902d19dc3148e7f8ae2103e051dd3573daa05964487c93fe5a5b37b76fe94729c8c2b372845f5d85e0722c53ae
```

是如下的形式。

```
0 [签名 1] [签名 2] [赎回脚本的内容]
```

赎回脚本为

```
522102429adbb84a4a0f14b31c14f4927418207bcef7f70eb97b1caed49160733bff6921026ce3c7280d473b7a9eab8fe76219687deb646c1619ad18902d19dc3148e7f8ae2103e051dd3573daa05964487c93fe5a5b37b76fe94729c8c2b372845f5d85e0722c53ae
```

你可以用这个 [工具](https://www.fileformat.info/tool/hash.htm)，计算赎回脚本 HASH160 的 [值](https://www.fileformat.info/tool/hash.htm?hex=3f2dc7046d9a969c401d78081f82aaa299b0b095f4ef1776212cdf873d2da546)，看看是不是 `8bd55244e4f86fb631e908f8cd9d9084e6744ad1`。

对照 [操作码说明书](https://en.bitcoin.it/wiki/Script#Opcodes)，翻译赎回脚本的内容。

```
52  <== 2
21  <== 接下来的 0x21 字节是数据
02429adbb84a4a0f14b31c14f4927418207bcef7f70eb97b1caed49160733bff69  <== 33 字节的数据
21  <== 接下来的 0x21 字节是数据
026ce3c7280d473b7a9eab8fe76219687deb646c1619ad18902d19dc3148e7f8ae  <== 33 字节的数据
21  <== 接下来的 0x21 字节是数据
03e051dd3573daa05964487c93fe5a5b37b76fe94729c8c2b372845f5d85e0722c  <== 33 字节的数据
53  <== 3
ae  <== OP_CHECKMULTISIG
```

是不是有点眼熟？这个赎回脚本，正是

```
2 [公钥 1] [公钥 2] [公钥 3] 3 OP_CHECKMULTISIG
```

解码后的解锁脚本为

```
 0 [签名 1] [签名 2]  2 [公钥 1] [公钥 2] [公钥 3] 3 OP_CHECKMULTISIG

|<------ 参数 ----->|<----------------- 赎回脚本 ------------------>|
```

你能看到，

> A P2SH transaction locks the output to this hash instead of the longer redeem script

P2SH 将输出锁定到脚本哈希，而不是锁定到特别长的具体脚本。

> Instead of "pay to this 5-key multisignature script," the P2SH equivalent transaction is "pay to a script with this hash."

用 P2SH 实现多签，只需要告诉付款方赎回脚本的哈希，取代 “向 N 个多重签名的具体脚本支付”，等同于 “向有该哈希值的脚本支付”。

P2SH 让付款到复杂脚本变得跟 P2PKH 一样简单。

- 向付款方提供脚本哈希，就像 P2PKH 需要公钥哈希一样
- 赎回脚本的内容，从锁定脚本转移到了解锁脚本中，更多的交易费也从发送方转移到收款方

目前网络中的大部分 P2SH 交易都是多签交易，** 但 P2SH 拥有更广泛的可能性 **，你可以在 P2SH 的赎回脚本中充分发挥想象力。

## One more thing

P2PK、P2PKH、old-style multisig 和 P2SH 都是网络支持的标准交易类型。

需要注意的是，考虑到安全因素，比特币网络默认只传播（relay）这些标准交易。

> Note that there is a small number of standard script forms that are relayed from node to node; non-standard scripts are accepted if they are in a block, but nodes will not relay them.

在尝试交易脚本的无限可能前，你应该先读一读 [源码](https://github.com/bitcoin-sv/bitcoin-sv/tree/master/src/policy) 中的 policy 部分。

## 参考

- 精通比特币（第二版）[译文](https://wizardforcel.gitbooks.io/masterbitcoin2cn/content/) [原文](https://github.com/bitcoinbook/bitcoinbook/)
- [Bitcoin - multisig Scripts](https://webbtc.com/scripts/multisig)
- [The state of Bitcoin multisig](https://medium.com/@alcio/the-state-of-bitcoin-multisig-82b3bf09b1ca)
- [Multi Sig Use Cases](https://github.com/bitpay/copay/wiki/Multi-Sig-Use-Cases)
- [Pay to script hash](https://en.bitcoin.it/wiki/Pay_to_script_hash)
- [Bitcoin multisig the hard way: Understanding raw P2SH multisig transactions](https://www.soroushjp.com/2014/12/20/bitcoin-multisig-the-hard-way-understanding-raw-multisignature-bitcoin-transactions/)
- [p2sh.info – Latest stats on Bitcoin transaction types](https://p2sh.info/dashboard/db/home-dashboard)
- [P2SH outputs statistics](https://p2sh.info/dashboard/db/p2sh-statistics)
- [P2SH repartition by type](https://p2sh.info/dashboard/db/p2sh-repartition-by-type)
- [bitcore library API #Script](https://bitcore.io/api/lib/script)
- [Script](https://en.bitcoin.it/wiki/Script)
