# P2PK 和 P2PKH

## 付款到公钥

构造交易时，如果 ** 锁定脚本中关联的是收款人的公钥 **，我们称这笔交易，是一笔 ** 付款到公钥 **（P2PK，Pay to Public Key）的交易。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/OXLqNXK.png)

对交易 `6880c2ae2f417bf6451029b716959ed255f127eeba3d7b40cf0ff3c44c5980c2`，

```
01000000
01  <== 交易有 1 个输入
0cb06c52c251cd97a9427611e7356d5705b135327df475e1e4eed7287c5191a4
06000000
6b
483045022100d2fb1e6e099f3572ed7f89659e2366f9b5fcb44552bd83e31b2ebb95c4f4a50f022034904e5cea8affb2cc886d480a9a0dbed9dd42ead138f8cbb7a2c5fc82ce1cb001210335b7e991f239f9097d9f7e5b0d115b6a4983d4d38066016a12b65c5cf8b33eba
01000000
01  <== 交易有 1 个输出
04be020000000000
23
21030e7061b9fb18571cf2441b2a7ee2419933ddaa423bc178672cd11e87911616d1ac
00000000
```

输出上的锁定脚本为

```
21030e7061b9fb18571cf2441b2a7ee2419933ddaa423bc178672cd11e87911616d1ac
```

根据 [操作码说明书](https://en.bitcoin.it/wiki/Script#Opcodes)，翻译一下，

```
21  <== 接下来的 0x21 字节是数据
030e7061b9fb18571cf2441b2a7ee2419933ddaa423bc178672cd11e87911616d1  <== 这是 33 字节的数据
ac  <== 操作码 OP_CHECKSIG
```

得到锁定脚本的内容为

```
030e7061b9fb18571cf2441b2a7ee2419933ddaa423bc178672cd11e87911616d1 OP_CHECKSIG
```

是不是有点眼熟？脚本第一项以 `0x03` 开头的 33 字节的数据，正是用压缩格式表示的公钥。

看 [这篇文章](https://en.bitcoin.it/wiki/OP_CHECKSIG) 了解 `OP_CHECKSIG` 操作码的工作方式。

这个 UTXO 在交易 `aa2cb91301d684c443f81db2a623baed5ea657ea33c5fc4d8f995033292ebbd6` 中被消费，下面是这笔交易的数据。

```
01000000
01
c280594cc4f30fcf407b3dbaee27f155d29e9516b7291045f67b412faec28068
00000000
49
48304502210096bf18f3fce1f3ac800f98c12e655240cbdb7127e046a3c1bd72e16d3e16c3fd022030e897e2eea143eca4e87b28ba104625eb316f1a487a72c5aecc93b824c8c74f01
ffffffff
01
7b24020000000000
19
76a9143b08b0003ef9b4f37f8d18a54f9db8ec67f8cbf088ac00000000
```

用同样的方法，找到这个 UTXO 的解锁脚本，也就是对应私钥的数字签名为

```
304502210096bf18f3fce1f3ac800f98c12e655240cbdb7127e046a3c1bd72e16d3e16c3fd022030e897e2eea143eca4e87b28ba104625eb316f1a487a72c5aecc93b824c8c74f01
```

验证交易时，只需将解锁脚本和锁定脚本连起来，执行即可。

```
[签名] [公钥] OP_CHECKSIG
```

为了说清楚交易的内幕细节，我们一直都在手工解码交易的原始数据，虽然有些麻烦，但这能帮助我们更好的理解比特币协议的工作方式。

可以直接使用区块链浏览器，来查询任意一笔交易的内容，解码原始交易数据，关联交易链条。

* 注意，在各种应用程序中看到的很多信息字段，实际上并不存在于比特币系统中 *。

用 [Blockchair](https://blockchair.com/bitcoin/transaction/6880c2ae2f417bf6451029b716959ed255f127eeba3d7b40cf0ff3c44c5980c2) 查询刚才的交易 `6880c2ae2f417bf6451029b716959ed255f127eeba3d7b40cf0ff3c44c5980c2`，能看到交易的详细信息。

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/CpmDms2.png)</div>

交易的原始数据（Raw transaction）为

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/eHjnpVd.png)</div>

解码后的交易（Decoded transaction）为

```
{
   "txid":"6880c2ae2f417bf6451029b716959ed255f127eeba3d7b40cf0ff3c44c5980c2",
   "hash":"6880c2ae2f417bf6451029b716959ed255f127eeba3d7b40cf0ff3c44c5980c2",
   "version":1,
   "size":202,
   "vsize":202,
   "weight":808,
   "locktime":0,
   "vin":[
      {
         "txid":"a491517c28d7eee4e175f47d3235b105576d35e7117642a997cd51c2526cb00c",
         "vout":6,
         "scriptSig":{
            "asm":"3045022100d2fb1e6e099f3572ed7f89659e2366f9b5fcb44552bd83e31b2ebb95c4f4a50f022034904e5cea8affb2cc886d480a9a0dbed9dd42ead138f8cbb7a2c5fc82ce1cb0[ALL] 0335b7e991f239f9097d9f7e5b0d115b6a4983d4d38066016a12b65c5cf8b33eba",
            "hex":"483045022100d2fb1e6e099f3572ed7f89659e2366f9b5fcb44552bd83e31b2ebb95c4f4a50f022034904e5cea8affb2cc886d480a9a0dbed9dd42ead138f8cbb7a2c5fc82ce1cb001210335b7e991f239f9097d9f7e5b0d115b6a4983d4d38066016a12b65c5cf8b33eba"
         },
         "sequence":1
      }
   ],
   "vout":[
      {
         "value":0.00179716,
         "n":0,
         "scriptPubKey":{
            "asm":"030e7061b9fb18571cf2441b2a7ee2419933ddaa423bc178672cd11e87911616d1 OP_CHECKSIG",
            "hex":"21030e7061b9fb18571cf2441b2a7ee2419933ddaa423bc178672cd11e87911616d1ac",
            "reqSigs":1,
            "type":"pubkey",
            "addresses":[
               "1CjRf1RMrTwyGoBHDbqzXERhVFkPyowt8i"
            ]
         }
      }
   ]
}
```

## 付款到公钥哈希

构造交易时，如果 ** 锁定脚本中关联的是收款人公钥的哈希 **，我们称这笔交易，是一笔 ** 付款到公钥哈希 **（P2PKH，Pay to Public Key Hash）的交易。

哈希运算是单向运算，隐藏原来的数据，得到其 ** 数据指纹 **。

公钥可以被当成身份标识，所以公钥的哈希，也可以作为一种身份标识。

比特币在计算公钥的哈希时，使用 HASH160 方法。

- 先对公钥做 SHA256 运算，得到 S1
- 再对 S1 做 RIPEMD160 运算，得到结果

Alice 从 Joe 那里，用现金换取了一些比特币，交易是 `7957a35fe64f80d234d76d83a2a8f1a0d8149a41d81de548f0a65a8a999f6f18`。

```
{
   "txid":"7957a35fe64f80d234d76d83a2a8f1a0d8149a41d81de548f0a65a8a999f6f18",
   "hash":"7957a35fe64f80d234d76d83a2a8f1a0d8149a41d81de548f0a65a8a999f6f18",
   "version":1,
   "size":225,
   "vsize":225,
   "weight":900,
   "locktime":0,
   "vin":[
      {
         "txid":"713eef22615ffb7c2f8f813e79a0d1e170d05a99218e291c33daca258f284d52",
         "vout":0,
         "scriptSig":{
            "asm":"3046022100a59e516883459706ac2e6ed6a97ef9788942d3c96a0108f2699fa48d9a5725d1022100f9bb4434943e87901c0c96b5f3af4e7ba7b83e12c69b1edbfe6965f933fcd17d[ALL] 04e5a0b4de6c09bd9d3f730ce56ff42657da3a7ec4798c0ace2459fb007236bc3249f70170509ed663da0300023a5de700998bfec49d4da4c66288a58374626c8d",
            "hex":"493046022100a59e516883459706ac2e6ed6a97ef9788942d3c96a0108f2699fa48d9a5725d1022100f9bb4434943e87901c0c96b5f3af4e7ba7b83e12c69b1edbfe6965f933fcd17d014104e5a0b4de6c09bd9d3f730ce56ff42657da3a7ec4798c0ace2459fb007236bc3249f70170509ed663da0300023a5de700998bfec49d4da4c66288a58374626c8d"
         },
         "sequence":4294967295
      }
   ],
   "vout":[
      {
         "value":0.1,
         "n":0,
         "scriptPubKey":{
            "asm":"OP_DUP OP_HASH160 7f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a8 OP_EQUALVERIFY OP_CHECKSIG",
            "hex":"76a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac",
            "reqSigs":1,
            "type":"pubkeyhash",
            "addresses":[
               "1Cdid9KFAaatwczBwBttQcwXYCpvK8h7FK"
            ]
         }
      }
   ]
}
```

这笔交易创建的 UTXO 上的锁定脚本为

```
OP_DUP OP_HASH160 7f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a8 OP_EQUALVERIFY OP_CHECKSIG
```

之后，Alice 去 Bob 的咖啡店购买咖啡，交易是 `0627052b6f28912f2703066a912ea577f2ce4da4caa5a5fbd8a57286c345c2f2`。

```
{
   "txid":"0627052b6f28912f2703066a912ea577f2ce4da4caa5a5fbd8a57286c345c2f2",
   "hash":"0627052b6f28912f2703066a912ea577f2ce4da4caa5a5fbd8a57286c345c2f2",
   "version":1,
   "size":258,
   "vsize":258,
   "weight":1032,
   "locktime":0,
   "vin":[
      {
         "txid":"7957a35fe64f80d234d76d83a2a8f1a0d8149a41d81de548f0a65a8a999f6f18",
         "vout":0,
         "scriptSig":{
            "asm":"3045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e3813[ALL] 0484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adf",
            "hex":"483045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e381301410484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adf"
         },
         "sequence":4294967295
      }
   ],
   "vout":[
      {
         "value":0.015,
         "n":0,
         "scriptPubKey":{
            "asm":"OP_DUP OP_HASH160 ab68025513c3dbd2f7b92a94e0581f5d50f654e7 OP_EQUALVERIFY OP_CHECKSIG",
            "hex":"76a914ab68025513c3dbd2f7b92a94e0581f5d50f654e788ac",
            "reqSigs":1,
            "type":"pubkeyhash",
            "addresses":[
               "1GdK9UzpHBzqzX2A9JFP3Di4weBwqgmoQA"
            ]
         }
      },
      {
         "value":0.0845,
         "n":1,
         "scriptPubKey":{
            "asm":"OP_DUP OP_HASH160 7f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a8 OP_EQUALVERIFY OP_CHECKSIG",
            "hex":"76a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac",
            "reqSigs":1,
            "type":"pubkeyhash",
            "addresses":[
               "1Cdid9KFAaatwczBwBttQcwXYCpvK8h7FK"
            ]
         }
      }
   ]
}
```

消费这个 UTXO 的解锁脚本为

```
3045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e3813[ALL]
0484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adf
```

是不是有点眼熟？解锁脚本第二项以 `0x04` 开头的 65 字节的数据，正是用不压缩格式表示的公钥。

P2PKH 的解锁脚本中除了包含数字签名，还包含公钥。

验证交易的脚本为

```
[签名] [公钥] OP_DUP OP_HASH160 [公钥的哈希] OP_EQUALVERIFY OP_CHECKSIG
```

下图模拟了这类脚本的执行过程，不再赘述，操作码的定义请自行查阅 [说明书](https://en.bitcoin.it/wiki/Script#Opcodes)。

<div style="width: 50%; margin: auto">![](https://github.com/bitcoinbook/bitcoinbook/raw/develop/images/mbc2_0605.png)</div>

<div style="width: 50%; margin: auto">![](https://github.com/bitcoinbook/bitcoinbook/raw/develop/images/mbc2_0606.png)</div>

注意到，签名的最后一个字节 `0x01`，被解码成了标记 `[ALL]`，它标识了签名的具体类型（SIGHASH，Signature Hash Types）。

你发现没有，P2PKH 跟 P2PK，没有本质区别。

验证一个数字签名是否有效，需要三个参数：

- 消息
- 公钥
- 数字签名

交易中签名的消息，是交易本身，更准确的说，是交易中特定数据子集的哈希（通过 SIGHASH 标记区分）。

P2PK 将公钥放在锁定脚本中，所以解锁脚本只需要提供数字签名。

P2PKH 交易的锁定脚本中，用公钥的哈希隐藏了公钥本身，所以在解锁脚本中，除了要提供数字签名，还要提供公钥。验证时，除了要验证数字签名，还要验证 ** 解锁脚本中的公钥 ** 是不是跟 ** 锁定脚本中的公钥哈希 ** 相匹配。

你可以用这个 [工具](https://www.fileformat.info/tool/hash.htm)，计算公钥

```
0484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adf
```

HASH160 的 [结果](https://www.fileformat.info/tool/hash.htm?hex=98f8648abb4e1333afac93709deae013c59c72f745de8f09041ff5295493b001)，看看是不是

```
7f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a8
```

## One more thing

网络中的绝大多数交易，都是 P2PKH。

P2PK 也是合法的交易类型，比 P2PKH 更简单，但出于安全方面的考虑，已不再被推荐使用。

> Pay to public key scripts are a simplified form of the p2pkh but aren't commonly used in new transactions anymore because p2pkh scripts are more secure (the public key is not revealed until the output is spent).

下面是关于 P2PK 和 P2PKH 的更多资料。

- [IP-to-IP payments](https://en.bitcoin.it/wiki/IP_transaction)
- [[pull] Remove send to IP address and IP transactions support](https://bitcointalk.org/index.php?topic=9334.0)
- [Why does the default miner implementation use pay-to-pubkey?](https://bitcoin.stackexchange.com/questions/32639/why-does-the-default-miner-implementation-use-pay-to-pubkey/32642)
- [Why does some coinbase scripts not check public key hash?](https://bitcointalk.org/index.php?topic=515724.0)
- [Does P2PKH substitute P2PK in any circumstances? Why?](https://bitcoin.stackexchange.com/questions/63699/does-p2pkh-substitute-p2pk-in-any-circumstances-why)
- [Why is P2PKH used instead of the simpler P2PK?](https://bitcoin.stackexchange.com/questions/72184/why-is-p2pkh-used-instead-of-the-simpler-p2pk)

# 参考

- 精通比特币（第二版）[译文](https://wizardforcel.gitbooks.io/masterbitcoin2cn/content/) [原文](https://github.com/bitcoinbook/bitcoinbook/)
- [learn me a bitcoin, P2PK](https://learnmeabitcoin.com/glossary/p2pk)
- [Bitcoin P2PKH Transaction Breakdown](https://medium.com/coinmonks/bitcoin-p2pkh-transaction-breakdown-bb663034d6df)
- [bitcore library API #Script](https://bitcore.io/api/lib/script)
