# 时间锁

时间锁功能让比特币交易拥有了时间维度，这篇文章介绍时间锁的详细内容。

本文也是介绍比特币交易内幕细节的最后一篇文章。

## 区块

为了理解时间锁，需要提前介绍一点区块的概念。

你可以把区块（Block）想象成一个箱子，里面装着交易。

对这些 “箱子”：

- 可以按先来后到的顺序，用 ** 高度 ** 标识，第一个箱子的高度为 0
- 也可以用哈希标识，相当于这个箱子的标签
- 每个箱子，都记录了它 ** 前一个 ** 箱子的标签（哈希）

网络不断产生新的交易，人们根据规则，将这些交易 “塞到” 新箱子里，随后将它摞在之前整理好的最后一个箱子上。

容易想象，网络中的交易，被不断整理到箱子中，这些依次摞起来的箱子，形成了一条 ** 不断延长 ** 的 “箱子”** 链 **。

比特币网络的总帐本，就是这样的一条链，交易填充区块，一个个区块 ** 前向引用 **，形成 ** 区块链 **。

## nLocktime

回忆一下比特币交易的结构。

长度（字节） | 描述
--- | ---
 4 | 交易结构的版本
 1~9 VarInt | 交易包含几个输入，非零正整数
 变长 | 输入数组
 1~9 VarInt | 交易包含几个输出，非零正整数
 变长 | 输出数组
 4 | nLockTime

通过最后 4 字节的 `nLockTime` 字段，可以实现 ** 交易 ** 粒度的时间锁。

- `nLockTime = 0`，表示这笔交易没有时间锁定，可以被 “随时” 写入账本，“即时生效”
- `nLockTime < 500,000,000`，指示块 ** 高度 **，这笔交易在块高度 `nLockTime` 之后，才可以被写入账本
- `nLockTime >= 500,000,000`，指示具体的 **Unix 时间戳 **，这笔交易在 Unix 时间戳 `nLockTime` 之后，才可以被写入账本

Unix 时间戳是一种时间表示法，它的值，表示从 `1970 年 1 月 1 日 0 时 0 分 0 秒 UTC` 开始，经历的秒数。

你可以用这个 [工具](https://www.epochconverter.com/) 转换，`1546434313` 表示的时间为 `2019 年 1 月 2 日 13 时 5 分 13 秒 UTC`，即北京时间 `2019 年 1 月 2 日 21 时 5 分 13 秒 UTC+8`。

设想这样的需求，父亲想立个遗嘱，在自己去世后，儿子可以拥有自己所有的比特币，并需要保证：

- 父亲在世时，可以随时修改遗嘱
- 父亲过世后，儿子确定可以拿到币

设置交易的 `nLockTime` 字段，可以满足这样的遗嘱需求。

1. 父亲计算出自己 80 岁时的 Unix 时间戳，值为 `T`
2. 父亲构造一笔 P2PKH 交易，将自己所有的比特币，付款到儿子的公钥哈希，并设置交易 `nLockTime` 字段的值为 `T`
3. 父亲用自己的私钥，对这笔交易签名（设置正确的解锁脚本），将签名后的交易数据交给儿子

这笔交易是合法的，但因为时间锁的设置，即使向网络 “展示” 这笔交易，它也不会被提前写入账本。转账不会在父亲 80 岁前发生，即儿子不会在父亲 80 岁前拿到这笔钱。

如果父亲去世时没到 80 岁，儿子也可以在未来，在父亲 80 岁这天之后，向网络 “展示” 交易，拿到这笔钱。

如果父亲想修改遗嘱，只需要：

1. 将所有比特币先转到自己的另一个公钥哈希上，即时生效
2. 重新按照自己的意愿，构造交易并设置 `nLockTime`，签名后分发

完成操作 [1] 后，之前那笔签名过的交易会 ** 变得无效 **，因为对应的 UTXO 已经被消费。

一般的，Alice 签名了一笔交易，付款到 Bob 的公钥哈希，并将交易的 `nLocktime` 设为三个月后。Alice 把这笔交易发送给 Bob，此时，两人都知道：

- 在三个月过去之前，Bob 不会收到这笔钱
- 这三个月内，Alice 可以随时构造另外的交易，即时生效，消费同样的 UTXO，即 Alice 可以在这三个月内，随时花费付给 Bob 的这笔钱
- Bob 无法保证 Alice 不这么做

这正是 `nLocktime` 的局限性。`nLocktime` 唯一能保证的，是这笔 ** 交易在时间锁释放之前 **，** 无法被写入账本 **，即收款人无法在时间锁释放之前，收到资金。

交易粒度的 `nLocktime` 时间锁，只在下列情况满足时，才会释放。

- `nLocktime = 0`，没有时间锁
- `nLockTime < 500,000,000`，且当前的区块高度，已经超过了 `nLockTime` 的值
- `nLockTime >= 500,000,000`，且当前的 Unix 时间戳，已经超过了 `nLockTime` 的值

## OP_CHECKLOCKTIMEVERIFY

为了改善交易 `nLocktime` 时间锁的局限性，有更细粒度的控制，时间锁需要跟 UTXO 关联，即放到锁定脚本中。

2015 年 12 月，[BIP-65](https://github.com/bitcoin/bips/blob/master/bip-0065.mediawiki) 引入操作码 `OP_CHECKLOCKTIMEVERIFY`（CLTV，Check Lock Time Verify），来实现 UTXO 粒度的时间锁定。

对一般的 P2PKH，其锁定脚本为

```
OP_DUP OP_HASH160 [公钥哈希] OP_EQUALVERIFY OP_CHECKSIG
```

如果一个 UTXO 的锁定脚本形如

```
 [过期时间] OP_CHECKLOCKTIMEVERIFY OP_DROP  OP_DUP OP_HASH160 [公钥哈希] OP_EQUALVERIFY OP_CHECKSIG

|<--------------CLTV 时间锁 --------------->|
```

我们说，这是一个被 `CLTV` 锁定的 UTXO，只能在锁定脚本中的 `CLTV` 时间锁释放后才可以被消费。

其中，`[过期时间]` 与交易的 `nLockTime` 字段有相同的格式，指示一个区块高度（`< 500,000,000`），或一个 Unix 时间戳（`>= 500,000,000`）。

即，只有在当前区块高度超过 `[过期时间]`，或当前 Unix 时间戳超过 `[过期时间]` 时，`CLTV` 时间锁才会释放，这个 UTXO 才可以被消费。

## CLTV 的幕后细节

逻辑上 `CLTV` 很好理解，但其工作方式稍显复杂，具体在 [BIP-65](https://github.com/bitcoin/bips/blob/master/bip-0065.mediawiki) 里定义，这里也做个说明。

回忆一下交易输入最后 4 字节的 `nSequence` 字段，这里会用到。

长度（字节） | 描述
--- | ---
 32 | 引用的交易哈希，UTXO 来自哪笔交易
 4 | 引用的输出序号，UTXO 是那笔交易的第几个输出，从 `0` 开始计数
 1~9 VarInt| 后面紧跟的解锁脚本，有多少字节
 变长 | 解锁脚本的内容
 4 | nSequence

如果一笔交易要消费 `CLTV` 锁定的 UTXO，需要同时满足下列所有条件。

- 这笔交易（消费这个 UTXO 的）输入的 `nSequence` 字段的值，必须小于 `0xffffffff`
- 这个 UTXO 锁定脚本中 `[过期时间]` 的值，必须大于等于 `0`
- 这笔交易的 `nLockTime` 和这个 UTXO 锁定脚本中的 `[过期时间]`，必须同时大于等于 `500,000,000` 或同时小于 `500,000,000`，即要么都指示 Unix 时间戳，要么都指示区块高度
- ** 这笔交易的 `nLockTime` 字段的值，必须大于等于这个 UTXO 锁定脚本中 `[过期时间]` 的值 **

你能看到，交易要消费 `CLTV` 锁定的 UTXO，还需要配合使用 `nLockTime` 字段。

这些条件组合有些复杂，与当前区块高度或当前 Unix 时间戳，好像并没有什么关系。

让我们换个角度看，着重关注最后一个条件。对一笔交易，

- 为了保证交易能 “即时生效”，你需要将 `nLockTime` 的值，设置为小于等于当前的区块高度或小于等于当前的 Unix 时间戳，否则这笔交易不会被写入账本
- 为了满足上述最后一个条件，你需要将 `nLockTime` 的值，设置为大于等于锁定脚本中 `[过期时间]` 的值，否则时间锁不会释放

画个图，直观的看一下。如果当前时间 `Tc`，未到 `CLTV` 锁定的过期时间 `Tb`：

<div style="width: 50%; margin: auto">![](https://www.lucidchart.com/publicSegments/view/5d145f0e-021f-4b5a-acc5-5493815f57d8/image.png)</div>

那么，你找不到这么一个 `nLockTime` 值 `T`，同时满足 `T <= Tc`（交易即时生效）且 `T>= Tb`（释放时间锁）。

如果当前时间 `Tc`，超过了 `CLTV` 锁定的过期时间 `Tb`：

<div style="width: 50%; margin: auto">![](https://www.lucidchart.com/publicSegments/view/d26de9f5-62ae-46b6-917c-8aa510ab6bbf/image.png)</div>

那么，`nLockTime` 可以被设置为大于等于 `Tb` 且小于等于 `Tc` 的任意值，一般的，都直接设置为当前的区块高度或当前的 Unix 时间戳 `Tc`。

对于

- 使用 `nLockTime` 字段的交易时间锁
- 使用 `CLTV` 操作码的 UTXO 时间锁

这两种时间锁都是 ** 指定某个具体的绝对时间点 ** 为过期时间的 ** 绝对时间 ** 锁。

## nSequence

`nSequence` 字段最初被设计为，标识某些还未被写入账本（仍在内存池中）的交易，允许它们在之后被更新。

- 如果某个交易的输入，其 `nSequence` 字段的值，小于 `0xffffffff`，表示这笔交易尚未 “确定”，还不是最终版本
- 这笔交易会被暂时搁置，等待被另一个消耗了同样输入的，并且有一个更大 `nSequence` 值的交易替换
- 直到收到消耗了同样输入的、`nSequence` 值为 `0xffffffff` 的交易，才认为这笔交易已经准备就绪，可以随时被写入账本

但这个功能没有实现，从未在系统中使用过。

[BIP-68](https://github.com/bitcoin/bips/blob/master/bip-0068.mediawiki) 通过复用交易输入的 `nSequence` 字段，实现 ** 交易 ** 粒度的 ** 相对时间 ** 锁。

相对时间的意思是，从 ** 被引用的交易写入账本 ** 后，经过的时间。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/OXLqNXK.png)

- 高度 10,000 的区块中有一笔交易，创建了一个 UTXO
- 你现在创建一笔新交易消费这个 UTXO，并设置新交易输入 `nSequence` 字段的值为 100 个区块

那么，这笔新交易不会被写入账本，除非当前区块高度已经达到或超过 10,100。

`nSequence` 是输入的一个字段，交易可以包含多个输入。

只有在满足了 ** 所有输入 ** 上的 `nSequence` 相对时间锁（如果有）要求后，** 交易 ** 才被认为合法，才会被写入账本。

根据 [BIP-68](https://github.com/bitcoin/bips/blob/master/bip-0068.mediawiki) 的设计，如果 `nSequence` 字段的值小于 $2^{31}$（`0x80000000`），表示这是一笔激活了 `nSequence` 相对时间锁的交易。

用 `nSequence` 表示相对时间锁的过期时间，格式上与 `nLockTime` 和 `CLTV` 略有不同。

对于这个 4 字节的值，

![](https://raw.githubusercontent.com/bitcoinbook/bitcoinbook/develop/images/mbc2_0701.png)

- 最高位第 31 位，作为开关，为 `1` 表示 ** 禁用 ** 相对时间锁
- 第 22 位作为类型标志，为 `1` 指示 ** 多少个 512 秒 **，为 `0` 指示 ** 多少个区块 **
- 低 16 位，作为值

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/OXLqNXK.png)

如果 `nSequence` 的值为 `0x0040005a`，

```
 31         22       15                0
 |          |        |                 |
 0000 0000 0100 0000 0000 0000 0101 1010
```

可以知道：

- 第 31 位是 `0`，启用了相对时间锁
- 第 22 位是 `1`，指示多少个 512 秒
- 低 16 位是 `0x005a`，值为 90
- 超时时间是 $90 \times 512 = 46080$ 秒

整理下 `nSequence` 的不同情况：

- 等于 $2^{32} - 1$，即等于 `0xffffffff`，表示没有设置任何时间锁
- 小于 $2^{32} - 1$ 且 大于等于 $2^{31}$，表示启用了 `nLockTime` 或 `CLTV` 绝对时间锁，一般都设置为 `0xfffffffe`
- 小于 $2^{31}$，即小于 `0x80000000`，表示启用了 `nSequence` 相对时间锁

## OP_CHECKSEQUENCEVERIFY

通过 [BIP-112](https://github.com/bitcoin/bips/blob/master/bip-0112.mediawiki) 引入的操作码 `OP_CHECKSEQUENCEVERIFY`（CSV，Check Sequence Verify），可以实现 UTXO 粒度的相对时间锁定。

锁定脚本形如

```
 [过期时间] OP_CHECKSEQUENCEVERIFY OP_DROP  OP_DUP OP_HASH160 [公钥哈希] OP_EQUALVERIFY OP_CHECKSIG

|<---------------CSV 时间锁 --------------->|
```

简单的说，被 `CSV` 锁定的 UTXO，从 ** 创建它的交易被写入账本 ** 开始计时，只有在经过一定的秒数或一定的区块数后，这个 UTXO 才可以被消费。

与 `CLTV` 类似，在消费一个 `CSV` 锁定的 UTXO 时，要求满足下列所有条件。

- 交易的 `nVersion` 字段的值，必须大于等于 `2`
- 输入的 `nSequence` 字段的值，必须小于 `0x800000000`
- 锁定脚本中 `[过期时间]` 的值，必须大于等于 `0` 且小于 `0x800000000`
- 输入的 `nSequence` 和锁定脚本中的 `[过期时间]`，要么都指示秒数，要么都指示区块数
- ** 输入的 `nSequence` 的值，必须大于等于 `[过期时间]` 的值 **

## 总结

这篇文章介绍了四种比特币交易时间锁：

- 基于交易粒度的 `nLockTime` 绝对时间锁，在达到指定的区块高度或具体的 Unix 时间戳前，这笔交易不会被写入账本
- 基于交易粒度的 `nSequence` 相对时间锁，这笔交易不会被写入账本，除非其输入引用的那笔已经被写入账本的交易，经过了指定的时间或区块数
- 基于 UTXO 粒度的 `CLTV` 绝对时间锁，在达到指定的区块高度或具体的 Unix 时间戳前，这笔 UTXO 无法被消费
- 基于 UTXO 粒度的 `CSV` 相对时间锁，在创建这个 UTXO 的交易写入帐本后，除非经过了指定的时间或区块数，否则这个 UTXO 无法被消费

关于 Median Time-Past 和 Fee Sniping，因为需要理解区块和挖矿的相关内容，所以我把他们放在之后的文章中再介绍。

时间锁的概念非常好理解，但细节繁多略显晦涩，建议你在阅读本文的同时，也看看下面的参考资料。

## 参考

- 精通比特币（第二版）[译文](https://wizardforcel.gitbooks.io/masterbitcoin2cn/content/) [原文](https://github.com/bitcoinbook/bitcoinbook/)
- [Bitcoin wiki, Protocol documentation #tx](https://en.bitcoin.it/wiki/Protocol_documentation#tx)
- [Bitcoin wiki, Script #Locktime](https://en.bitcoin.it/wiki/Script#Locktime)
- [比特币的时间锁转账](https://www.jianshu.com/p/5bd02d5ca4a6)
- [Transactions with a wait time (using nLockTime)](https://bitcoin.stackexchange.com/questions/5783/transactions-with-a-wait-time-using-nlocktime)
- [Understanding Timelock Options](https://github.com/ChristopherA/Learning-Bitcoin-from-the-Command-Line/blob/master/09_1_Understanding_Timelock_Options.md)
- [Using CLTV in Scripts](https://github.com/ChristopherA/Learning-Bitcoin-from-the-Command-Line/blob/master/09_2_Using_CLTV_in_Scripts.md)
- [Using CSV in Scripts](https://github.com/ChristopherA/Learning-Bitcoin-from-the-Command-Line/blob/master/09_3_Using_CSV_in_Scripts.md)
