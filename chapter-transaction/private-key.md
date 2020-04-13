# 比特币私钥

## 私钥

比特币的私钥（k）是一个 ** 随机数 **，可以是 $1$ 和 $(n - 1)$ 之间的任何数字，其中 $n$ 是常数 $1.158 \times 10^{77}$，略小于 $2^{256}$。

生成一个私钥，本质上就是选择一个数。只要选取的方法不可预测或不可重复，它就是密码学安全的。

我们可以用不同的表示法，来表示同一个私钥。

格式 | 值
--- | ---
十六进制原始数据（Raw） | f97c89aaacf0cd2e47ddbacc97dae1f88bec49106ac37716c451dcdd008a4b62
WIF 不压缩格式（WIF uncompressed） | 5KiANv9EHEU4o9oLzZ6A7z4xJJ3uvfK2RLEubBtTz1fSwAbpJ2U
WIF 压缩格式（WIF compressed） | L5agPjZKceSTkhqZF2dmFptT5LFrbr6ZGPvP7u4A6dvhTrr71WZ9

WIF 是 Wallet Import Format 的缩写，有压缩和不压缩两种格式，都是 Raw 格式的私钥经过 Base58Check 编码之后的结果。

在比特币系统中，大多数需要向用户展示的数据都使用 Base58Check 编码。它可以压缩数据，提高可读性，避免歧义，并且包含错误校验，能有效防止数据在转录过程中产生错误。

## Base58Check 编码

> Base64 编码使用了 26 个小写字母、26 个大写字母、10 个数字以及 2 个符号（+ 和 /），用于在电子邮件这样的基于文本的媒介中传输二进制数据。

[Base58](https://zh.wikipedia.org/wiki/Base58) 是 Base64 编码格式的子集，舍弃了一些容易错读和在特定字体中容易混淆的字符，不使用 Base64 中的 `0`（数字 0）、`O`（大写字母 o）、`l`（小写字母 L）、`I`（大写字母 i）以及 `+` 和 `/`，是比特币系统中使用的一种独特的编码方式。

Base58Check 在编码过程中加入了校验和，在最后使用 Base58，是一种 ** 可逆编码 **。对于要编码的数据 payload：

1. 在 payload 前添加版本前缀 Version ，得到 S1
2. 对 S1 做两次 SHA256 哈希运算得到 S2，取 S2 的前 4 字节作为校验和 Checksum
3. 在 S1 后附上 Checksum，得到 S3
4. 对 S3 做 Base58 编码，得到结果

![](https://raw.githubusercontent.com/bitcoinbook/bitcoinbook/develop/images/mbc2_0406.png)

对于不同类型的数据，在做 Base58Check 编码时会添加不同的版本前缀，以产生易于辨识的不同结果。

类型 | 版本前缀的值（十六进制） | Base58Check 之后的前缀
---|---|---
P2PKH 地址 | 00 | 1
P2SH 地址 | 05 | 3
测试网络（testnet）地址 | 6F | m 或 n
WIF 格式的私钥 | 80 | 5，K 或 L
根据 BIP-38 标准 加密的私钥 | 0142 | 6P
根据 BIP-32 标准定义的 扩展公钥 | 0488B21E | xpub

## WIF

为了方便用户导入私钥，定义了 WIF ，分为不压缩和压缩两种类型。

WIF 是 Base58Check 编码之后的结果，根据上面的表格，WIF 在编码时使用的版本前缀为 `0x80`。

对 Raw 格式的私钥 `f97c89aaacf0cd2e47ddbacc97dae1f88bec49106ac37716c451dcdd008a4b62`，计算其 **WIF 不压缩 ** 格式：

```
// 私钥本身就是 payload
f97c89aaacf0cd2e47ddbacc97dae1f88bec49106ac37716c451dcdd008a4b62

// 下面是 Base58Check 定义的过程

// 给 payload 添加版本前缀 0x80，得到 S1
80 f97c89aaacf0cd2e47ddbacc97dae1f88bec49106ac37716c451dcdd008a4b62

// 对 S1 做两次 SHA256，得到 S2
701ccdd192515bf36a241b9fca879d7915a458cfb36ebcf2c8db1d796dc63b4a

// 取 S2 的前 4 字节，得到 Checksum
701ccdd1

// S3 = S1 + Checksum
80 f97c89aaacf0cd2e47ddbacc97dae1f88bec49106ac37716c451dcdd008a4b62 701ccdd1

// WIF_uncompressed = Base58 (S3)，结果以 5 开头
5KiANv9EHEU4o9oLzZ6A7z4xJJ3uvfK2RLEubBtTz1fSwAbpJ2U
```

稍加改变，在私钥后添加一个压缩标志位，计算结果就是 **WIF 压缩 ** 格式。

```
// 在私钥后添加 压缩标志后缀 0x01，得到 payload
f97c89aaacf0cd2e47ddbacc97dae1f88bec49106ac37716c451dcdd008a4b62 01

// 下面是 Base58Check 定义的过程

// 给 payload 添加版本前缀 0x80，得到 S1
80 f97c89aaacf0cd2e47ddbacc97dae1f88bec49106ac37716c451dcdd008a4b62 01

// 对 S1 做两次 SHA256，得到 S2
14372b9cd0d344b679ac30ab70000c245c3c7888907449bccd0caf830a84c2ed

// 取 S2 的前 4 字节，得到 Checksum
14372b9c

// S3 = S1 + Checksum
80 f97c89aaacf0cd2e47ddbacc97dae1f88bec49106ac37716c451dcdd008a4b62 01 14372b9c

// WIF_uncompressed = Base58 (S3)，结果以 K 或 L 开头
L5agPjZKceSTkhqZF2dmFptT5LFrbr6ZGPvP7u4A6dvhTrr71WZ9
```

WIF 压缩和不压缩两种格式都使用同样的版本前缀，唯一的区别就是，是否在私钥后添加压缩标志后缀。

你可以使用下面这些工具，体验转换的过程。

- [不压缩 WIF 与 Raw 格式私钥的转换工具](https://gobittest.appspot.com/PrivateKey)
- [十六进制 Hash 工具](https://www.fileformat.info/tool/hash.htm)
- [十六进制 Base58 编码工具](https://incoherency.co.uk/base58/)

## 椭圆曲线密码学（ECC，Elliptic Curve Cryptography）

要理解比特币公钥的计算细节，需要先了解一些 ECC 的背景知识。

ECC 是一种 [非对称加密算法](https://zhuanlan.zhihu.com/p/101907402)，基于椭圆曲线上的离散对数数学问题。

$y^2 = x^3 - x + 1$ 定义了一条椭圆曲线，函数图像如下。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/0MGgi5G.png)

比特币使用的椭圆曲线，为

$$y^2 = x^3 + 7 \pmod p$$

这条椭圆曲线由 [Secp256k1](https://en.bitcoin.it/wiki/Secp256k1) 标准定义，其中

- mod 是取模运算（取余数），$1 = 5 \pmod 2$
- p 是定值 $2^{256} - 2^{32} - 2^9 - 2^8 - 2^7 - 2^6 - 2^4 - 1$，这是一个非常大的素数

因为方程两端取模运算的存在，所以这条曲线的函数图像并不是连续的，而是二维空间上一系列散开的点。

如果 $p = 17$，它的函数图像如下。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/79rRrpL.png)

你可以把 Secp256k1 的函数图像，想象成一个 ** 极大的网格上一系列更为复杂的散点 **。

椭圆曲线上的两点 p1 和 p2，定义 $p = p1 + p2$ 为椭圆曲线上 ** 点的加法运算 **。

1. 过点 p1 和 p2 做直线与椭圆曲线相交点 q
2. 以 X 轴对称翻转点 q，得到点 p

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/5FstmO4.png)

如果 p1 和 p2 为同一个点，则过 p1 和 p2 的连线，变成了过该点的椭圆曲线的切线。使用 [这个工具](https://www.desmos.com/calculator/ialhd71we3)，可以有更多可视化的直观体验。

乘法的定义可以从加法扩展，其中 k 为整数。

$$k \times p = \underbrace{p + p + ... + p}\_{k}$$

下图展示了从椭圆曲线上的点 G，计算 2G、4G、8G 的操作。

![](https://raw.githubusercontent.com/bitcoinbook/bitcoinbook/develop/images/mbc2_0404.png)

* 注意，为了有直观的理解，图片中的椭圆曲线都是连续的。比特币使用的 Secp256k1 曲线，其图像并不连续，但它们的数学原理是相同的。*

## 公钥

比特币的公钥（K），是 **Secp256k1 定义的椭圆曲线上的一个点 **。

$$K = k \times G$$

其中，k 为私钥（一个整数），G 为椭圆曲线上的一个固定点，这个点由 Secp256k1 标准定义，它的坐标（十六进制表示）为

```
G.x = 79BE667E F9DCBBAC 55A06295 CE870B07 029BFCDB 2DCE28D9 59F2815B 16F81798
G.y = 483ADA77 26A3C465 5DA4FBFC 0E1108A8 FD17B448 A6855419 9C47D08F FB10D4B8
```

比特币的私钥（k）和公钥（K）之间的关系是固定的，数学原理保证 ** 计算过程单向不可逆，能轻而易举的从私钥计算出其对应的公钥，反过来则无法实现 **。

刚才的例子中，私钥 `f97c89aaacf0cd2e47ddbacc97dae1f88bec49106ac37716c451dcdd008a4b62`，其对应的公钥为

```
x = e46dcd7991e5a4bd642739249b0158312e1aee56a60fd1bf622172ffe65bd789
y = 97693d32c540ac253de7a3dc73f7e4ba7b38d2dc1ecc8e07920b496fb107d6b2
```

公钥也有两种表示法，压缩格式和不压缩格式。

在公钥坐标前添加前缀 `0x04`，可以直接得到不压缩格式的公钥。

```
04 e46dcd7991e5a4bd642739249b0158312e1aee56a60fd1bf622172ffe65bd789 97693d32c540ac253de7a3dc73f7e4ba7b38d2dc1ecc8e07920b496fb107d6b2
```

为了表示不压缩格式的公钥，需要 65 字节，1 字节前缀，32 字节 X 坐标，32 字节 Y 坐标。

根据 Secp256k1 曲线的特点，如果知道公钥 X 坐标的值和 Y 坐标的奇偶，就可以直接推算出其 Y 坐标的值。定义压缩格式的公钥：

- 如果 Y 坐标的值为偶数，在 X 坐标前添加前缀 `0x02`
- 如果 Y 坐标的值为奇数，在 X 坐标前添加前缀 `0x03`

<div style="width: 50%; margin: auto">![](https://raw.githubusercontent.com/bitcoinbook/bitcoinbook/develop/images/mbc2_0407.png)</div>

得到压缩格式的公钥为

```
02 e46dcd7991e5a4bd642739249b0158312e1aee56a60fd1bf622172ffe65bd789
```

为了表示压缩格式的公钥，需要 33 字节，1 字节前缀，32 字节 X 坐标。

你可以参考下面的代码，自己试一试。

```go
package main

import (
    "encoding/hex"
    "fmt"
    "github.com/decred/dcrd/dcrec/secp256k1"
)

func main() {
    privateKeyBytes, _ := hex.DecodeString("f97c89aaacf0cd2e47ddbacc97dae1f88bec49106ac37716c451dcdd008a4b62")
    _, publicKey := secp256k1.PrivKeyFromBytes(privateKeyBytes)
    fmt.Println("pk.x = " + hex.EncodeToString(publicKey.X.Bytes()))
    fmt.Println("pk.y = " + hex.EncodeToString(publicKey.Y.Bytes()))

    fmt.Println("uncompressed public key = " + hex.EncodeToString(publicKey.SerializeUncompressed()))
    fmt.Println("compressed public key = " + hex.EncodeToString(publicKey.SerializeCompressed()))
}

// 输出
// pk.x = e46dcd7991e5a4bd642739249b0158312e1aee56a60fd1bf622172ffe65bd789
// pk.y = 97693d32c540ac253de7a3dc73f7e4ba7b38d2dc1ecc8e07920b496fb107d6b2
// uncompressed public key = 04e46dcd7991e5a4bd642739249b0158312e1aee56a60fd1bf622172ffe65bd78997693d32c540ac253de7a3dc73f7e4ba7b38d2dc1ecc8e07920b496fb107d6b2
// compressed public key = 02e46dcd7991e5a4bd642739249b0158312e1aee56a60fd1bf622172ffe65bd789
```

## 总结

- 比特币的私钥，本质是一个数
- 选取私钥的过程如果不可预测或不可重复（是随机的），则私钥是密码学安全的
- 私钥可以用 WIF 压缩或 WIF 不压缩格式表示，WIF 压缩格式的私钥前缀是 `K` 或 `L`，WIF 不压缩格式的私钥前缀是 `5`
- WIF 格式使用了 Base58Check 编码，这是一个 ** 可逆编码 **，它包含版本前缀、数据块和校验和三部分，容易识别，方便转录
- ECC 是一种基于椭圆曲线数学问题的非对称加密算法，比特币使用 Secp256k1 标准定义的一条特殊的椭圆曲线，它的函数图像是一堆复杂的散点
- 比特币的公钥，本质是 Secp256k1 曲线上的一个点，由其对应的私钥计算得出
- 椭圆曲线的数学特性保证，从比特币私钥计算其对应公钥是一个单向运算，无法通过公钥计算出其对应的私钥
- 公钥可以用压缩和不压缩两种格式表示，压缩格式的公钥前缀是 `0x02` 或 `0x03`，不压缩格式的公钥前缀是 `0x04`
- 没有 “压缩的私钥” 和 “压缩的公钥”，“压缩” 只是针对其表示方法，而不是针对私钥和公钥本身

你可能注意到，公钥用压缩格式表示时，有效降低了存储空间，但 WIF 压缩和不压缩格式，其结果从长度上看并没有明显的区别。

为什么要这么做？接下来的文章中说。

## 参考

- [椭圆曲线密码学](https://zh.wikipedia.org/wiki/% E6% A4% AD% E5%9C%86% E6%9B% B2% E7% BA% BF% E5% AF%86% E7% A0%81% E5% AD% A6)
- [椭圆曲线加密算法](https://juejin.im/post/5a67f3836fb9a01c9b661bd3)
- [ECC 椭圆曲线加密算法：介绍](https://zhuanlan.zhihu.com/p/36326221)
- [SEC 2: Recommended Elliptic Curve Domain Parameters](https://www.secg.org/sec2-v2.pdf)
- [How to convert Private Key WIF Compressed start with a 'L' to Private Key WIF starts with a '5'](https://www.reddit.com/r/Bitcoin/comments/7fptly/how_to_convert_private_key_wif_compressed_start/)
- [package secp256k1](https://godoc.org/github.com/decred/dcrd/dcrec/secp256k1)
- 精通比特币（第二版）[译文](https://wizardforcel.gitbooks.io/masterbitcoin2cn/content/) [原文](https://github.com/bitcoinbook/bitcoinbook/)
