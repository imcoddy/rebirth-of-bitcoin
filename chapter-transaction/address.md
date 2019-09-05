# 比特币地址

在接收比特币时，通常会使用一串跟银行卡号类似的 “神奇代码”，作为收款 “地址”，形如 `1BJhat1AMGYbT9HYJxVekoCaPaqB9ZyTyF`。例如，当 Alice 向 Bob 支付比特币的时候，如果她：

- 使用 P2PK 交易，需要知道 **Bob 的公钥 **，锁定脚本为 `[Bob 的公钥] OP_CHECKSIG`
- 使用 P2PKH 交易，需要知道 **Bob 的公钥 ** 或 **Bob 的公钥哈希 **，锁定脚本为 `OP_DUP OP_HASH160 [Bob 的公钥哈希] OP_EQUALVERIFY OP_CHECKSIG`
- 使用 P2SH 交易，需要知道 **Bob 的脚本哈希 **，锁定脚本为 `OP_HASH160 [Bob 的脚本哈希] OP_EQUAL`

本节将介绍地址是怎么计算的。


## P2PKH(Pay-to-Pubkey-Hash)

一句话解释，P2PKH 地址是 Base58Check 编码的公钥哈希。

我们用 App 支付比特币到某个地址，App 会先将地址反解成公钥哈希，然后构造一笔 P2PKH 交易，完成支付。

因为 Base58Check 编码是 **可逆** 的，所以可以这么做。

用 https://www.bitaddress.org/ 生成一对私钥和地址，整体演算一遍。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/FCahVtj.png)

私钥 `L2M6qFzrdDEv7eRT7p94Zf4QShLdWjicFpkiCro12CrRbCDb8ke3` 以 `L` 开头，这是 WIF 压缩格式的私钥。

用这个 [工具](https://incoherency.co.uk/base58/)，Base58Check 反解之后为（我加了空格）

```
80 98fd2a819a382f8e142e38242f6caf2a2f6f58e7fe6ca5f23c5b0818b15b4ba6 01 236803e8
```

私钥为 `98fd2a819a382f8e142e38242f6caf2a2f6f58e7fe6ca5f23c5b0818b15b4ba6`。

计算出对应的公钥点为

```
x = da52d817a5ae3555f36a94528322eb47016f1334b798f5b4fa614a892dabb3ea
y = f314bf38816673c55c1708cf1e36b55c936db97618d6460c4d223be83bec7788
```

公钥为 `02da52d817a5ae3555f36a94528322eb47016f1334b798f5b4fa614a892dabb3ea`（P2PK 交易需要这个值）。

用这个 [工具](https://www.fileformat.info/tool/hash.htm)，计算公钥 HASH160 后的 [结果](https://www.fileformat.info/tool/hash.htm?hex=b4327ae841ca6cfd1203cc6e135a48d893cbfb2c388c0f64a79694199cc9b4cb)，得到公钥哈希（P2PKH 交易需要这个值）为

```
710a35053db4296dc5f3476d10ae93924bc55c9a
```

对 ** 公钥哈希 ** 做 Base58Check 编码，按规定添加 `0x00` 版本前缀。

类型 | 版本前缀的值（十六进制） | Base58Check 之后的前缀
---|---|---
P2PKH 地址 | 00 | 1
P2SH 地址 | 05 | 3

得到地址

```
1BJhat1AMGYbT9HYJxVekoCaPaqB9ZyTyF
```

编码过程如下。

```
# payload
710a35053db4296dc5f3476d10ae93924bc55c9a

# S1 = version + payload
00 710a35053db4296dc5f3476d10ae93924bc55c9a

# S2 = SHA256(SHA256(S1))
5f91f9a69fc02d537c93e41b127b2f5081fd25f82683c1eebe3c54aae1f83130

# S2 的前 4 字节是 Checksum
5f91f9a6

# S3 = S1 + Checksum
00 710a35053db4296dc5f3476d10ae93924bc55c9a 5f91f9a6

# 地址 = Base58 (S3)
1BJhat1AMGYbT9HYJxVekoCaPaqB9ZyTyF
```

需要注意的是，从 P2PKH 地址只能反解出公钥哈希的值，而无法知道公钥本身，因为 ** 哈希是不可逆 ** 的，无法从公钥哈希计算公钥。

由于在 Base58Check 编码时加入了特定前缀 `0x00`，所以这类地址都以 `1` 开头，方便识别。

地址中包含了校验和，能有效防止转录过程中的错误，这也是为什么不直接使用公钥哈希的原因。

当你在 App 里发送比特币到某个 `1` 开头的地址时，App 知道这是一个 P2PKH 地址，先校验地址是不是合法，再反解出编码前的公钥哈希，创建 P2PKH 交易，完成支付。

普通用户可以直接使用地址来收发比特币，而不用了解幕后细节，因为它们被地址隐藏了。


## P2PK(Pay-to-Public-Key)
P2PK 锁定版脚本形式如下：

```
<Public Key A> OP_CHECKSIG
```

用于解锁的脚本是一个简单签名：

```
<Signature from Private Key A>
```
经由交易验证软件确认的组合脚本为：

```
<Signature from Private Key A> <Public Key A> OP_CHECKSIG
```
根据上方的规则去运行就可以发现，此规则比 P2PKH 要简单的多，只有一步验证，少了上方的地址验证。其实，P2PKH 被创建主要目的一方面为使比特币地址更简短，使之更方便使用，核心内容还是 P2PK 的。

## P2SH(Pay-to-Script-Hash)

同样的，对于 P2SH 交易中用到的脚本哈希，也可以这么干。

但编码时，会使用 `0x05` 的版本前缀，所以 P2SH 地址都以 `3` 开头，形如 `347N1Thc213QqfYCz3PZkjoJpNv5b14kBd`。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/OXLqNXK.png)

交易 `40eee3ae1760e3a8532263678cdf64569e6ad06abc133af64f735e52562bccc8`，

```
{
   "txid":"40eee3ae1760e3a8532263678cdf64569e6ad06abc133af64f735e52562bccc8",
   "hash":"40eee3ae1760e3a8532263678cdf64569e6ad06abc133af64f735e52562bccc8",
   "version":1,
   "size":189,
   "vsize":189,
   "weight":756,
   "locktime":0,
   "vin":[
      {
         "txid":"42a3fdd7d7baea12221f259f38549930b47cec288b55e4a8facc3c899f4775da",
         "vout":0,
         "scriptSig":{
            "asm":"3044022048d1468895910edafe53d4ec4209192cc3a8f0f21e7b9811f83b5e419bfb57e002203fef249b56682dbbb1528d4338969abb14583858488a3a766f609185efe68bca[ALL] 031a455dab5e1f614e574a2f4f12f22990717e93899695fb0d81e4ac2dcfd25d00",
            "hex":"473044022048d1468895910edafe53d4ec4209192cc3a8f0f21e7b9811f83b5e419bfb57e002203fef249b56682dbbb1528d4338969abb14583858488a3a766f609185efe68bca0121031a455dab5e1f614e574a2f4f12f22990717e93899695fb0d81e4ac2dcfd25d00"
         },
         "sequence":4294967295
      }
   ],
   "vout":[
      {
         "value":0.0099,
         "n":0,
         "scriptPubKey":{
            "asm":"OP_HASH160 e9c3dd0c07aac76179ebc76a6c78d4d67c6c160a OP_EQUAL",
            "hex":"a914e9c3dd0c07aac76179ebc76a6c78d4d67c6c160a87",
            "reqSigs":1,
            "type":"scripthash",
            "addresses":[
               "3P14159f73E4gFr7JterCCQh9QjiTjiZrG"
            ]
         }
      }
   ]
}
```

输出支付到了脚本哈希 `e9c3dd0c07aac76179ebc76a6c78d4d67c6c160a`，让我们计算一下对应的地址。

```
# payload
e9c3dd0c07aac76179ebc76a6c78d4d67c6c160a

# S1 = version + payload
05 e9c3dd0c07aac76179ebc76a6c78d4d67c6c160a

# S2 = SHA256(SHA256(S1))
# http://bit.ly/2RoiDBu
7f297f3156a14d114af23b33c339817a1241afcb52a154d19683c67c538b7278

# S2 的前 4 字节是 Checksum
7f297f31

# S3 = S1 + Checksum
05 e9c3dd0c07aac76179ebc76a6c78d4d67c6c160a 7f297f31

# 地址 = Base58 (S3)
# https://incoherency.co.uk/base58/
3P14159f73E4gFr7JterCCQh9QjiTjiZrG
```

同样的，从 P2SH 地址只能反解出脚本哈希的值，而无法知道赎回脚本的具体内容。

当你在 App 里发送比特币到某个 `3` 开头的地址时，App 知道这是一个 P2SH 地址，先校验地址是不是合法，再反解出编码前的脚本哈希，创建 P2SH 交易，完成支付。

# 总结

地址是公钥哈希或脚本哈希的 Base58Check 编码，这个编码是可逆的。

P2PKH 地址都以 `1` 开头，P2SH 地址都以 `3` 开头。

结合之前的文章，你知道：

- 在比特币系统里其实没有 “地址” 的概念，交易输出会被锁定脚本锁定，与公钥哈希或脚本哈希关联
- 地址是公钥哈希或脚本哈希的可逆编码，定义 “地址” 是为了方便用户使用，封装幕后细节，简化操作
- 从私钥可以计算公钥，从公钥可以计算地址，但反过来都不成立
- 私钥十分重要，拥有私钥就拥有了其对应公钥上锁定的比特币

![](https://github.com/bitcoinbook/bitcoinbook/raw/develop/images/mbc2_0401.png)

# 参考

- 精通比特币（第二版）[译文](https://wizardforcel.gitbooks.io/masterbitcoin2cn/content/) [原文](https://github.com/bitcoinbook/bitcoinbook/)
