# 比特币交易的结构

比特币的交易，由一个或多个输出和一个或多个输入（Coinbase 交易是一种特殊情况）构成。

交易的每个输出上，都会附上一个加密难题，定义将来在花费这笔 UTXO 时需要满足的条件。

交易的每个输入上，都要提供一个解锁脚本，解决或满足之前附在这笔 UTXO 上的加密难题或条件，解锁 UTXO 用于支付。

如果你从前面的文章一路看过来，理解了比特币交易的细节，你应该能设计出下面的数据结构。

对交易的每个输出 TxOut，需要有

- 这个 UTXO 的币值
- 锁定脚本

对交易的每个输入 TxIn，需要有

- 这笔 UTXO 来自之前哪笔交易的第几个输出（需要表达 ** 交易链条 **）
- 解锁脚本

对交易，需要有

- 这笔交易的哈希（数据指纹），用于标识和索引这笔交易
- TxIn 数组，表示这笔交易的所有输入
- TxOut 数组，表示这笔交易的所有输出

这样的设计能满足需求，同时又足够精简。这篇文章，介绍比特币交易的数据结构。

<!-- more -->

# 输出

[代码](https://github.com/bitcoin-sv/bitcoin-sv/blob/d9b12a23db/src/primitives/transaction.h#L153)

```cpp
/**
 * An output of a transaction.  It contains the public key that the next input
 * must be able to sign with to claim it.
 */
class CTxOut {
public:
    Amount nValue;         // UTXO 的币值，8 字节整数，单位是聪
    CScript scriptPubKey;  // 锁定脚本

    ......
};
```

`Amount` 的类声明在 [这里](https://github.com/bitcoin-sv/bitcoin-sv/blob/d9b12a23db/src/amount.h#L16)。

```cpp
struct Amount {
private:
    int64_t amount;

    ......
};
```

`CScript` 的类声明在 [这里](https://github.com/bitcoin-sv/bitcoin-sv/blob/d9b12a23db/src/script/script.h#L404)。

# 输入

[代码](https://github.com/bitcoin-sv/bitcoin-sv/blob/d9b12a23db/src/primitives/transaction.h#L79)

```cpp
/**
 * An input of a transaction. It contains the location of the previous
 * transaction's output that it claims and a signature that matches the output's
 * public key.
 */
class CTxIn {
public:
    COutPoint prevout;  // UTXO 的来源，包含一个交易哈希和一个索引号，用来表示哪笔交易的第几个输出
    CScript scriptSig;  // 解锁脚本
    uint32_t nSequence;

    ......
};
```

`COutPoint` 的类声明在 [这里](https://github.com/bitcoin-sv/bitcoin-sv/blob/d9b12a23db/src/primitives/transaction.h#L36)。

```cpp
/**
 * An outpoint - a combination of a transaction hash and an index n into its
 * vout.
 */
class COutPoint {
private:
    TxId txid;  // 交易哈希
    uint32_t n; // 输出的序号

    ......
};
```

输入有一个 4 字节的 `nSequence` 字段，以后的文章中再说。

# 交易

[代码](https://github.com/bitcoin-sv/bitcoin-sv/blob/d9b12a23db/src/primitives/transaction.h#L249)

```cpp
/**
 * The basic transaction that is broadcasted on the network and contained in
 * blocks. A transaction can contain multiple inputs and outputs.
 */
class CTransaction {
public:
    const int32_t nVersion;          // 交易结构的版本标识，4 字节
    const std::vector<CTxIn> vin;    // 输入数组
    const std::vector<CTxOut> vout;  // 输出数组
    const uint32_t nLockTime;

    ......

private:
    const uint256 hash;              // 交易的哈希

    ......
};
```

交易有一个 4 字节的 `nLockTime` 字段，以后的文章中再说。

# 序列化和反序列化

程序中，一般用特定的 ** 数据结构 **，来表示和存储具体的数据，就像上面描述的那样。

这样的数据，方便人们识别和理解，方便程序操作，但不方便在网络上传输。

在传输前需要将数据结构转换成方便网络传输的 ** 字节流 ** 形式，这个过程称为 [** 序列化 **](https://zh.wikipedia.org/wiki/% E5% BA%8F% E5%88%97% E5%8C%96)。

从字节流 “恢复” 数据成数据结构的形式，这个过程称为 ** 反序列化 **。

举个例子方便理解。我们可以定义下面的数据结构，来表示二十四小时制的时间。

```
Type Time {
    uint32_t hour;
    uint32_t minute;
    uint32_t second;
};
```

时、分、秒分别用 4 字节整数表示，`20:35:10` 可以表示为

```
Time t;
t.hour = 20;    // 00 00 00 14
t.minute = 35;  // 00 00 00 23
t.second = 10;  // 00 00 00 0a
```

注释后面是数据的十六进制表示。在传输数据时，发送

```
00 00 00 14 00 00 00 23 00 00 00 0a
```

并规定：

- 你会收到 12 字节的数据
- 第一个 4 字节数据是 “时”
- 第二个 4 字节数据是 “分”
- 第三个 4 字节数据是 “秒”

对方在收到数据后，就能根据规则，将字节流还原成数据结构的形式。

注意到，数据结构不仅包含数据的值，还描述 “这是什么数据”。

当你看到 `t.hour = 20`，你知道这个数据表示时间中的 “小时”，值为 20。

但当你看到 `00 00 00 14`，你只知道这个数据的值为 20，但不知道这是 20 时，还是 20 分，还是 20 秒。所以，需要定义序列化的规则。

另一个不容易注意到的点是，需要多字节表达的数据项（用 4 字节来表达 “小时” 字段）的值，如何在在字节流中排列。

上面的例子中，你收到第一个 4 字节的顺序为

```
00
00
00
14
```

你默认了，先收到的字节为这个数据的 ** 高位字节 **，后收到的为 ** 低位字节 **，所以你得到 `00 00 00 14`。

换个角度说，如果对方认为先收到的字节为这个数据的低位字节，那他会把这个数据解析成 `14 00 00 00`，引起错误。

所以，字节流传输时，还需要定义字节的排列模式，这是另一个很有意思的话题，称为 ** 字节序 **（Endianness），下面是一些资料和讨论。

- [理解字节序](https://www.ruanyifeng.com/blog/2016/11/byte-order.html)
- [Little Endian, The order of bytes that a computer like to read in.](https://learnmeabitcoin.com/glossary/little-endian)
- [What would you change about the Bitcoin protocol?](https://bitcointalk.org/index.php?topic=4278.0)

比特币系统中，** 除了解锁脚本和锁定脚本 **，其他部分均使用 ** 小端模式 ** 编码，认为 ** 先收到的字节为数据的低位字节 **。

如果我们以小端模式来传输刚才的数据，字节流应该是

```
14 00 00 00 23 00 00 00 0a 00 00 00
```

# 序列化输出

输出序列化后，格式如下。

长度（字节） | 描述
--- | ---
 8 | 以聪为单位的币值
 1~9 VarInt | 后面紧跟的锁定脚本，有多少字节
 变长 | 锁定脚本的内容

对于下面这个序列化后的交易输出，

```
60e31600000000001976a914ab68025513c3dbd2f7b92a94e0581f5d50f654e788ac
```

可以反序列化为

- `60e3160000000000` 是币值，小端模式，值为 `00 00 00 00 00 16 e3 60`，1500000 聪，0.015 比特币
- `19`，后面紧跟的 25 字节是锁定脚本
- `76a914ab68025513c3dbd2f7b92a94e0581f5d50f654e788ac`，锁定脚本的内容

# 序列化输入

输入序列化后，格式如下。

长度（字节） | 描述
--- | ---
 32 | 引用的交易哈希，UTXO 来自哪笔交易
 4  | 引用的输出序号，UTXO 是那笔交易的第几个输出，从 `0` 开始计数
 1~9 VarInt| 后面紧跟的解锁脚本，有多少字节
 变长 | 解锁脚本的内容
 4 | nSequence

对于下面这个序列化后的交易输入（我加了换行方便识别），

```
186f9f998a5aa6f048e51dd8419a14d8a0f1a8a2836dd734d2804fe65fa35779
00000000
8b
483045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e381301410484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adf
ffffffff
```

可以反序列化为

- 这个输入使用的 UTXO，是交易 `7957a35fe64f80d234d76d83a2a8f1a0d8149a41d81de548f0a65a8a999f6f18` 的第 0 个输出
- `8b`，后面紧跟的 139 字节，是解锁脚本，
- `48304502...8d752adf`，解锁脚本的内容
- `ffffffff` 是 `nSequence` 的值

# 序列化交易

交易由输入和输出构成。交易序列化后，格式如下。

长度（字节） | 描述
--- | ---
 4 | 交易结构的版本
 1~9 VarInt | 交易包含几个输入，非零正整数
 变长 | 输入数组
 1~9 VarInt | 交易包含几个输出，非零正整数
 变长 | 输出数组
 4 | nLockTime

你注意到没有，交易序列化后，** 没有 ** 交易哈希的部分。

只需要对序列化后的交易数据做哈希运算，就可以得到交易的哈希值，这种冗余的信息，并不需要传输。

通过下面的过程，计算交易的哈希。

1. 对序列化后的交易数据做 SHA256 运算，得到 S1
2. 对 S1 做 SHA256 运算，得到 S2
3. 按字节翻转 S2，得到交易的哈希

Alice 去 Bob 的咖啡店支付 0.015 比特币购买咖啡，生成了交易 `0627052b6f28912f2703066a912ea577f2ce4da4caa5a5fbd8a57286c345c2f2`。

![](https://github.com/bitcoinbook/bitcoinbook/raw/develop/images/mbc2_0204.png)

下面是这笔交易序列化后的样子（我替你加了换行），你能从中找到各个字段的信息吗？

```
01000000
01
186f9f998a5aa6f048e51dd8419a14d8a0f1a8a2836dd734d2804fe65fa35779
00000000
8b
483045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e381301410484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adf
ffffffff
02
60e3160000000000
19
76a914ab68025513c3dbd2f7b92a94e0581f5d50f654e788ac
d0ef800000000000
19
76a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac
00000000
```

点下面的链接，体验一下计算交易哈希的过程。

1. 对序列化后的交易数据做 SHA256，得到 [S1 的值](https://www.fileformat.info/tool/hash.htm?hex=0100000001186f9f998a5aa6f048e51dd8419a14d8a0f1a8a2836dd734d2804fe65fa35779000000008b483045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e381301410484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adfffffffff0260e31600000000001976a914ab68025513c3dbd2f7b92a94e0581f5d50f654e788acd0ef8000000000001976a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac00000000)`dda380359b9d149fbc48d95aebbbe59117d91fb19e00d13f8992b38ada9654be`
2. 第 S1 做 SHA256，得到 [S2 的值](https://www.fileformat.info/tool/hash.htm?hex=dda380359b9d149fbc48d95aebbbe59117d91fb19e00d13f8992b38ada9654be)`f2c245c38672a5d8fba5a5caa44dcef277a52e916a0603272f91286f2b052706`
3. 按字节翻转 S2，得到交易的哈希 `0627052b6f28912f2703066a912ea577f2ce4da4caa5a5fbd8a57286c345c2f2`

另一个需要注意的点是，Coinbase 交易虽然不需要输入，但结构上输入数组仍然存在（长度为 1），输入结构中的各字段也都会被置成特殊值，用来标识。

- 引用的交易哈希全为 `0`
- 引用的输出序号全为 `f`
- 解锁脚本的长度为 2 ~ 100 字节
- 解锁脚本的内容，随意
- `nSequence` 的值为 `ffffffff`

交易 `d0ec21e1d73d06be76c2b5b1e5ec486085bda8264229046c11b95f66f2eded83` 是一笔 Coinbase 交易，序列化后内容如下。

```
01000000
01  <== 输入数组的长度为 1
0000000000000000000000000000000000000000000000000000000000000000  <== 引用的交易哈希全为 0
ffffffff  <== 引用的输出序号全为 f
45
03ec59062f48616f4254432f53756e204368756e2059753a205a6875616e67205975616e2c2077696c6c20796f75206d61727279206d653f2f06fcc9cacc19c5f278560300
ffffffff
01
529c6d9800000000
19
76a914bfd3ebb5485b49a6cf1657824623ead693b5a45888ac
00000000
```

# One more thing

你有没有发现，序列化规则中，描述脚本长度、数组个数的字段，其长度也是变化的。

```
60e3160000000000 19 76a914ab68025513c3dbd2f7b92a94e0581f5d50f654e788ac
```

这是刚才的例子，前 8 字节 `60e3160000000000` 表示币值是确定的，因为规则定义了币值用 8 个字节表达。

但 “锁定脚本的大小” 字段，其长度是不确定的，可以用 1 ~ 9 个字节来表达。

# 参考

- 精通比特币（第二版）[译文](https://wizardforcel.gitbooks.io/masterbitcoin2cn/content/) [原文](https://github.com/bitcoinbook/bitcoinbook/)
- [Transaction](https://en.bitcoin.it/wiki/Transaction)
- [Protocol documentation #Variable length integer](https://en.bitcoin.it/wiki/Protocol_documentation#Variable_length_integer)
- [VarInt - A format for indicating the size of upcoming data](https://learnmeabitcoin.com/glossary/varint)
- [TxBinaryMap, Bitcoin Wiki](https://en.bitcoin.it/wiki/File:TxBinaryMap.png)
- [Coinbase Transaction, A transaction used to claim a block reward.](https://learnmeabitcoin.com/glossary/coinbase-transaction)
