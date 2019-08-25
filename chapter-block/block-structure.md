# 区块链

比特币的交易，会被 “整理” 到区块中，形成一条可追溯到创世块的链。

网络活动不断产生新交易，不断 “整理” 出新区块来记录 “这一段时间内” 的交易。

为了能彼此关联，每个区块都会记录它的前一个区块是什么，这相当于，区块按先来后到的顺序被 “摞” 在一起，形成了一条 “链”。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/rXwGgGZ.png)

一个区块，可以用高度标识，也可以用哈希标识。

随着时间的推移，链不断延长，这条 ** 区块链 **，记录了截至目前为止所有的比特币交易，是比特币网络的总账本。

我们说一笔交易被写入账本，就是说这笔交易通过了验证，已经被 “整理” 进区块，并得到了全网络的认可。

这篇文章，介绍比特币的区块和区块链。


## 区块的结构

当你托运行李的时候，航空公司会在你的箱上贴一个标签，记录一些必要的基本信息，方便快速识别。

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/nxjmrBV.png)</div>
用区块存储交易也是类似的，代码见 [block.h#L61](https://github.com/bitcoin-sv/bitcoin-sv/blob/d9b12a23db/src/primitives/block.h#L61)，区块结构（序列化后）为

长度（字节） | 描述
--- | ---
4 | 后面紧跟的区块数据，有多少字节
80 | 区块头
1~9 VarInt | 区块里有多少笔交易
变长 | 各个交易的数据

区块头就是这个区块的 “标签”，代码见 [block.h#L21](https://github.com/bitcoin-sv/bitcoin-sv/blob/d9b12a23db/src/primitives/block.h#L21)，区块头的结构（序列化后）为

长度（字节） | 字段 | 描述
------ |------- | ------
4 | nVersion | 版本号，用来跟踪软件或协议的升级
32 | hashPrevBlock | 前一个区块（父区块）的哈希值
32 | hashMerkleRoot | 一个哈希值，表示这个区块中全部交易构成的 Merkle 树的根
4 | nTime | 区块创建的时间，Unix 时间戳格式
4 | nBits | 难度目标，该区块工作量证明算法的难度目标
4 | nNonce | 一个用于证明工作量的计数器

你能看到，和交易一样，序列化后的区块结构中，也没有区块哈希的部分。

这个哈希标识，可以直接用收到的区块数据计算出来，并不需要传输。通过下面的过程，计算区块的哈希。

1. 对序列化后的 ** 区块头 ** 数据做 SHA256 运算，得到 S1
2. 对 S1 做 SHA256 运算，得到 S2
3. 按字节翻转 S2，得到区块的哈希

区块看起来会是下面的样子，黄色的部分是区块头。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/XZp5yPB.png)

高度 100,000 的区块，从 [序列化后的区块数据](https://blockchain.info/block/000000000003ba27aa200b1cecaad478d2b00432346c3f1f3986da1afd33e506?format=hex) 中找到区块头。

```
01000000  # nVersion
50120119172a610421a6c3011dd330d9df07b63616c2cc1f1cd0020000000000  # hashPrevBlock
6657a9252aacd5c0b2940996ecff952228c3067cc38d4885efb5a4ac4247e9f3  # hashMerkleRoot
37221b4d  # nTime
4c86041b  # nBits
0f2b5710  # nNonce
```

这个区块在 Unix 时间戳为 `0x4d1b2237` 时，即 `‭1293623863‬` 时被创建出来，[转换后的时间](https://www.epochconverter.com/) 为 `2010-12-29 11:57:43 UTC`，即北京时间 `2010-12-29 19:57:43 UTC+8`。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/CPsvad1.png)

1. 对区块头数据做 SHA256，得到 [S1 的值](https://www.fileformat.info/tool/hash.htm?hex=0100000050120119172a610421a6c3011dd330d9df07b63616c2cc1f1cd00200000000006657a9252aacd5c0b2940996ecff952228c3067cc38d4885efb5a4ac4247e9f337221b4d4c86041b0f2b5710)`00844eeb8713eb62bc33df34ca0cfa7af2ee152a6b16788fd3f2fea69861f3c8`
2. 对 S1 做 SHA256，得到 [S2 的值](https://www.fileformat.info/tool/hash.htm?hex=00844eeb8713eb62bc33df34ca0cfa7af2ee152a6b16788fd3f2fea69861f3c8)`06e533fd1ada86391f3f6c343204b0d278d4aaec1c0b20aa27ba030000000000`
3. 按字节翻转 S2，得到区块哈希 `000000000003ba27aa200b1cecaad478d2b00432346c3f1f3986da1afd33e506`

关于 `nBits` 和 `nNonce` 字段的含义，下篇文章介绍。

## Merkle 树

Merkle 树是一棵二叉树，用于 ** 归纳 ** 一个区块中的所有交易，代码见 [merkle.cpp](https://github.com/bitcoin-sv/bitcoin-sv/blob/d9b12a23db/src/consensus/merkle.cpp)。

Merkle 树会生成 ** 整个交易集合的数字指纹 **，形如

<div style="width: 50%; margin: auto">![](https://github.com/bitcoinbook/bitcoinbook/raw/develop/images/mbc2_0903.png)</div>
交易会被放在 Merkle 树最底层的叶子节点上，如图所示，如果交易的个数是奇数，会复制最后一笔交易补齐，其中：

- $H_A = SHA256(SHA256(Tx\ A))$
- $H_{AB} = SHA256(SHA256(H_A + H_B))$

$H_A + H_B$ 的意思拼接 $H_A$ 和 $H_B$，如果 $H_A = 0123$，$H_B = 4567$，则 $H_A + H_B = 01234567$。

不管区块中有多少交易，都使用 Merkle 树结构进行归纳，最顶上的树根 `Merkle Root` 的值，会放到区块头的 `hashMerkleRoot` 字段中。

Merkle 树将区块头和区块中的交易关联了起来，如果区块中的交易发生了变化，Merkle 树根的值就会变化，从而改变区块头，改变整个区块的哈希标识。

使用 Merkle 树的另一个好处是，它提供了一种校验区块是否存在某笔交易的高效途径。对于下面这棵 Merkle 树，

<div style="width: 50%; margin: auto">![](https://github.com/bitcoinbook/bitcoinbook/raw/develop/images/mbc2_0905.png)</div>
为了证明交易 K 在区块中，可以用 $H_L$、$H_{IJ}$、$H_{MNOP}$ 和 $H_{ABCDEFGH}$ 这四个哈希值构造一条 “Merkle 路径”，只需 128 字节，任何人都可以用这条路径，验证区块包含交易 K。

## Coinbase

一个区块最多只能有一笔 Coinbase 交易（一般是区块中的第一笔交易），它没有输入，“无中生有” 的产生输出，即会发行新的比特币。

伴随着区块的产生，新的比特币会被不断 “创造” 出来。

比特币协议规定：

- 每产生 210,000 个区块，新发行的比特币数量就会减半，直至为零
- 比特币的创世区块（高度为 0 的区块），发行 50 个比特币
- 网络会自动调整某些参数，以保证平均 10 分钟的区块生产速度
- 谁创建的区块，谁就获得这个区块中新发行的比特币（构造一个 Coinbase 交易放到区块中，输出到自己的地址），及区块中所有交易的手续费

产生 210,000 个区块，大约需要四年时间。

- 在比特币运行的第一个四年中，每个区块创造出 50 个新比特币
- 2012 年 11 月，比特币的发行速度降低到每区块 25 比特币
- 2016 年 7 月，降低到 12.5 比特币
- 2020 年的某个时候，也就是从区块高度 630,000 开始，它将再次下降至 6.25 比特币
- 直到第 6,720,000 块（大约在 2137 年产生），达到比特币的最小单位 1 聪
- 经过 6,930,000 个区块之后，大约在 2140 年，所有的共 `20999999.97690000` 比特币将全部发行完毕

关于比特币的总量，可以写个程序算一下。

```go
package main

import "fmt"

func main() {
    totalBitcoin := 0
    halfCount := 0 // 减半的次数
    for coinbase := 5000000000; coinbase >= 1; coinbase /= 2 {
        fmt.Println(coinbase)
        totalBitcoin += coinbase * 210000 // 每 210000 个区块 Coinbase 减半
        halfCount += 1
    }
    fmt.Println("------------------")
    fmt.Println(halfCount)
    fmt.Println(totalBitcoin)
}

// 输出
// 5000000000
// 2500000000
// 1250000000
// 625000000
// 312500000
// 156250000
// 78125000
// 39062500
// 19531250
// 9765625
// 4882812
// 2441406
// 1220703
// 610351
// 305175
// 152587
// 76293
// 38146
// 19073
// 9536
// 4768
// 2384
// 1192
// 596
// 298
// 149
// 74
// 37
// 18
// 9
// 4
// 2
// 1
// ------------------
// 33
// 2099999997690000
```

一个要注意的点是，因为 Coinbase 交易不需要输入，所以你可以在解锁脚本里填写任意值（2 ~ 100 字节），这个值被称为 Coinbase 数据。

根据 [BIP-34](https://github.com/bitcoin/bips/blob/master/bip-0034.mediawiki) 的要求，Coinbase 数据需要以这个 Coinbase 交易所在的区块高度开头。

交易 `d0ec21e1d73d06be76c2b5b1e5ec486085bda8264229046c11b95f66f2eded83` 的 Coinbase 数据为

```
03ec59062f48616f4254432f53756e204368756e2059753a205a6875616e67205975616e2c2077696c6c20796f75206d61727279206d653f2f06fcc9cacc19c5f278560300
```

开头的 `0x03` 表示，区块高度数据紧跟其后有 3 字节，值为 `0x0659ec`（小端模式），即 416236。

另外，比特币协议允许 Coinbase 交易发行新币的最大数量可以直观的说明剩余比特币的比例，比如在当前（2019 年）每个区块被允许发行新币的最大数量是 12.5，说明下次减半的时候，还有 12.5% 的比特币没有开采。

## 区块链

注意区块头中 `hashPrevBlock` 字段带来的神奇效果。

每个区块，都会将它前一个区块的哈希值写在自己的区块头中。

区块的哈希，是对区块头的哈希。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/x8gUkkP.png)

如果链中的某个区块数据发生了变化（例如区块中的交易被替换），那么这个区块的哈希也会发生变化（Merkle 树根发生了变化），同时改变下一个区块的区块头数据及区块哈希，带来一连串的连锁反应。

也就是说，从这个改变了的区块开始，之后所有的区块都必须重新计算。

这是比特币区块链一个非常重要的特性。

一段时间内产生的交易通过验证后，会被打包（整理）进区块并 “告知” 全网，网络中的节点根据规则，选择接受（跟随）这个区块（把这个区块纳入总帐本），或者拒绝（丢弃）这个区块。
