# HD 钱包

## 不确定性（随机）钱包

比特币的全节点软件一般都包含钱包功能，这种钱包只是随机生成私钥的集合（JBOK，Just a Bunch of Keys），称为不确定性钱包或随机钱包。

一个私钥，会对应唯一的公钥和地址，当你每次都使用新地址收发比特币时，会生成大量的私钥。

这些私钥之间彼此独立，毫无关联，意味着你需要经常备份用过的私钥，否则一旦钱包软件不可访问，你的比特币也会石沉大海。

[![](https://raw.githubusercontent.com/bitcoinbook/bitcoinbook/develop/images/mbc2_0501.png)](https://raw.githubusercontent.com/bitcoinbook/bitcoinbook/develop/images/mbc2_0501.png)

由于在备份和使用时过于麻烦，不确定性钱包已不再被推荐使用，逐渐被确定性钱包取代。

另外，全节点软件会下载所有的区块数据，如果你从头开始，这将是一个漫长的同步过程，通常需要几天时间，并占用掉几百 G 的硬盘空间。

从功能实现的角度看，钱包只需要 “知道” 那些与自己私钥有关的交易和区块便可正常工作，并不需要下载所有的区块数据。

现在的钱包软件基本都是开箱即用的，只需同步少量数据便可直接使用，十分方便。

## 确定性（种子）钱包

确定性钱包中的私钥都可以从一个随机种子（Seed）计算出来，计算过程是单向的，你无法从私钥计算出种子的内容。

这种钱包在备份和迁移时十分方便，备份一个种子就相当于备份了钱包中的所有私钥，向新钱包中导入种子就可以恢复所有私钥。

确定性钱包从逻辑上看，是下面的样子。

[![](https://raw.githubusercontent.com/bitcoinbook/bitcoinbook/develop/images/mbc2_0502.png)](https://raw.githubusercontent.com/bitcoinbook/bitcoinbook/develop/images/mbc2_0502.png)

## 分层确定性钱包

下列 BIP 共同定义了一种确定性钱包的实现，这种钱包被称为分层确定性（HD，Hierarchical Deterministic）钱包。

* [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)，HD 钱包中的密钥如何衍生
* [BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)，HD 钱包助记词（Mnemonic）和种子（Seed）的创建规则
* [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)，支持多币种和多账户的 HD 钱包

除此之外，还有

* [BIP-43](https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki)，多用途（purpose）HD 钱包的结构定义
* [BIP-45](https://github.com/bitcoin/bips/blob/master/bip-0045.mediawiki)，通过 P2SH 实现多签的 HD 钱包

“分层” 的意思是，钱包的中的私钥具有层级关系，[BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 定义了私钥间的树形结构。

“确定性” 的意思是，当种子（Seed）确定后，钱包中的所有私钥便都是确定的，都可以从这个种子计算出来，相同的种子计算出的私钥也都是相同的。

基于树形结构，HD 钱包的一个父密钥可以衍生出一系列子密钥，每个子密钥又可以继续衍生出一系列孙密钥，依次类推无限衍生下去，就像下图所示的样子。

[![](https://raw.githubusercontent.com/bitcoinbook/bitcoinbook/develop/images/mbc2_0503.png)](https://raw.githubusercontent.com/bitcoinbook/bitcoinbook/develop/images/mbc2_0503.png)

现在主流的钱包软件基本都是兼容 [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)、[BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) 和 [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) 的 HD 钱包。

### 从熵源创建助记词（Mnemonic Passphrase）

当你用 <https://www.bitaddress.org/> 生成一个私钥时，可以通过随意晃动鼠标和敲击键盘来引入更多的随机性。

钱包软件在创建私钥时，都需要引入类似这样的随机性以保证密码学上的私钥安全。

对 HD 钱包来说也是一样，** 一切计算工作都从一个随机序列开始 **。

1. 取一串有 LL 个二进制位的随机序列（熵）S1，LL 可以是 128、160、192、224 或 256
2. 对 S1 做 SHA256 运算，得到 S2
3. 把 S2 的高 L32L32 位作为校验和拼接在 S1 后，得到 S3，显而易见，S3 的长度可以被 11 整除

L+L32=33L32=11×3L32L+L32=33L32=11×3L32

1. 从高位到低位，将 S3 每 11 位划成一组，每组二进制序列都可以转换成一个十进制数，表示单词表中的行数
2. [BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) 定义的 [单词表](https://github.com/bitcoin/bips/blob/master/bip-0039/bip-0039-wordlists.md) 中有 2048 个单词，每个单词一行（行数从 0 开始计数），在表中查找这些行数上的单词
3. 将这些单词 ** 按顺序抄写下来 **，就是助记词

[![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/OXLqNXK.png)](https://aaron67-public.oss-cn-beijing.aliyuncs.com/OXLqNXK.png)

对下面 128 位的随机序列：

    5e5d507dc5d543a8c7415656dac4ba0c

计算助记词的过程为

    # S1
    5e5d507dc5d543a8c7415656dac4ba0c

    # S2 = SHA256(S1)
    # https://bit.ly/2Hv5Loe
    055a1b4dc3af3267362e5d89b707fac6a94ef40a7be5f20f0940de178f01ea33

    # 取高 4 位作为校验和
    # S3 = S1 + Checksum
    5e5d507dc5d543a8c7415656dac4ba0c 0

    # S3 的二进制串
    01011110010111010101000001111101110001011101010101000011101010001100011101000001010101100101011011011010110001001011101000001100 0000

    # 从高位到低位将 S3 每 11 位分成一组
    01011110010  # 754
    11101010100  # 1876
    00011111011  # 251
    10001011101  # 1117
    01010100001  # 673
    11010100011  # 1699
    00011101000  # 232
    00101010110  # 342
    01010110110  # 694
    11010110001  # 1713
    00101110100  # 372
    00011000000  # 192

从 [英语单词表](https://github.com/bitcoin/bips/blob/master/bip-0039/english.txt) 中查找这 12 个数对应的单词，得到这个随机序列对应的助记词为

    furnace tunnel buyer merry feature stamp brown client fine stomach company blossom

注意，

* 随机序列是 HD 钱包初始化的起点
* 助记词从随机序列计算得到，反过来，从助记词也可以计算出随机序列的内容

### 从助记词创建种子（Seed）

** 种子由助记词计算而来 **，使用 PBKDF2（Password-Based Key Derivation Function 2）方法。

1. PBKDF2 的第一个输入是助记词
2. PBKDF2 的第二个输入是盐（Salt），由 `mnemonic` 和用户指定的密语（Passphrase）拼接而成，** 这个密语是可选的 **
3. PBKDF2 使用 HMAC-SHA512 哈希算法，做 2048 次哈希运算来衍生输入，产生一个 512 位的输出，这个值就是 HD 钱包的种子

[![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/3XNaTDP.png)](https://aaron67-public.oss-cn-beijing.aliyuncs.com/3XNaTDP.png)

写一段程序从助记词计算种子。

```
    package main

    import (
        "encoding/hex"
        "fmt"
        "github.com/tyler-smith/go-bip39"
    )

    func main() {
        mnemonic := "furnace tunnel buyer merry feature stamp brown client fine stomach company blossom"
        fmt.Println(hex.EncodeToString(bip39.NewSeed(mnemonic, "")))
        fmt.Println(hex.EncodeToString(bip39.NewSeed(mnemonic, "bitcoin")))
    }

    // 输出
    // 2588c36c5d2685b89e5ab06406cd5e96efcc3dc101c4ebd391fc93367e5525aca6c7a5fe4ea8b973c58279be362dbee9a84771707fc6521c374eb10af1044283
    // 1e8340ad778a2bbb1ccac4dd02e6985c888a0db0c40d9817998c0ef3da36e846b270f2c51ad67ac6f51183f567fd97c58a31d363296d5dc6245a0a3c4a3e83c5
```

使用 [这个](https://github.com/crypto-browserify/pbkdf2) Node.js 库，可以看到 PBKDF2 的更多细节。
```
    const crypto = require('crypto');

    const hash = 'sha512'
    const round = 2048
    const seed_bytes = 64

    var mnemonic = 'furnace tunnel buyer merry feature stamp brown client fine stomach company blossom'
    var passphrase = ''
    var salt = 'mnemonic' + passphrase

    crypto.pbkdf2(mnemonic, salt, round, seed_bytes, hash, (err, derivedKey) => {
      if (err) throw err;
      console.log(derivedKey.toString('hex'));
    });
```

对于上面得到的助记词，在密语为空时，计算出的种子为

    2588c36c5d2685b89e5ab06406cd5e96efcc3dc101c4ebd391fc93367e5525aca6c7a5fe4ea8b973c58279be362dbee9a84771707fc6521c374eb10af1044283

如果密语为 `bitcoin`，计算出的种子为

    1e8340ad778a2bbb1ccac4dd02e6985c888a0db0c40d9817998c0ef3da36e846b270f2c51ad67ac6f51183f567fd97c58a31d363296d5dc6245a0a3c4a3e83c5

[![](https://www.lucidchart.com/publicSegments/view/eef20105-68c3-4220-ba3a-4f5111c5be94/image.png)](https://www.lucidchart.com/publicSegments/view/eef20105-68c3-4220-ba3a-4f5111c5be94/image.png)

你能看到，

* 种子从助记词和用户密语计算而来
* 助记词从一个随机序列计算而来，查阅特定的单词表后最终确定
* 即使随机序列的内容一样，查阅不同语言的单词表，可以得到不同的助记词，从而计算出不同的种子
* 即使助记词的内容一样，指定不同的密语，可以得到不同的种子

** HD 钱包的确定性来源于种子，当种子确定后，钱包中的所有私钥就都是确定的，都可以从种子计算出来 **。

所以你可以直接记录下这个种子的值，作为 HD 钱包的备份，只不过这一大串内容抄写起来有点麻烦。

对一个 HD 钱包，初始化种子的过程涉及到如下 ** 变量 **：

* 助记词（由随机序列的内容和助记词的语言共同决定）
* 用户指定的密语
* 币种的衍生路径

上面三个变量中，除了助记词是由系统硬件随机生成的之外，密语和衍生路径均可由用户设置或使用默认值。但在恢复钱包时如果有任何一个变量不同，则会被当成是一个新的种子。所以在备份 HD 钱包时，需要 ** 同时备份助记词和密语和衍生路径 **，这样的备份才是完整的。

HD 钱包中的私钥是树状的层级结构。

* 树根位置的私钥，称为主私钥（Master Private Key），从种子直接计算得到
* 树中的某个私钥，从其父私钥计算得到

## 从种子衍生主密钥

[BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 定义，HD 钱包使用 HMAC-SHA512 方法从种子衍生主私钥。

HMAC-SHA512 使用 SHA512 哈希算法，以一个消息（Message）和一个密钥（Key）作为输入，生成 512 位（64 字节）的消息摘要（Digest）作为输出。

从种子计算主私钥时，种子作为输入的消息，字符串 `Bitcoin seed` 作为输入的密钥，计算产生 512 位的输出。

* 输出的高 256 位，是主私钥
* 输出的低 256 位，是主链码（Master Chain Code）
```
    package main

    import (
        "crypto/hmac"
        "crypto/sha512"
        "encoding/hex"
        "fmt"
        "github.com/tyler-smith/go-bip39"
    )

    func main() {
        mnemonic := "furnace tunnel buyer merry feature stamp brown client fine stomach company blossom"
        seed := bip39.NewSeed(mnemonic, "")
        fmt.Println(hex.EncodeToString(seed))

        hmacSHA512 := hmac.New(sha512.New, []byte("Bitcoin seed"))
        hmacSHA512.Write(seed)
        digest := hmacSHA512.Sum(nil)
        fmt.Println("Master private key\t" + hex.EncodeToString(digest[:32]))
        fmt.Println("Master chain code\t" + hex.EncodeToString(digest[32:]))
    }

    // 输出
    // 2588c36c5d2685b89e5ab06406cd5e96efcc3dc101c4ebd391fc93367e5525aca6c7a5fe4ea8b973c58279be362dbee9a84771707fc6521c374eb10af1044283
    // Master private key   116c2daffad72d24cd3c122a65f937ec2743f98952e174ae158bf6ea70c78954
    // Master chain code    a74b75701aba81dd29e94226696cc0e67d6a5f29398d151c05c09c416dbf0865
```
对刚才的种子，计算出的主私钥为

    116c2daffad72d24cd3c122a65f937ec2743f98952e174ae158bf6ea70c78954

主链码为

    a74b75701aba81dd29e94226696cc0e67d6a5f29398d151c05c09c416dbf0865

这个从种子衍生出的主私钥，跟之前文章介绍的 [比特币私钥](https://aaron67.cc/2018/12/23/bitcoin-keys/) 没有任何区别，通过 Secp256k1 椭圆曲线乘法，可以计算出其对应的主公钥（Master Public Key）:

    02c8022cf8de6472f50f08b8b7e364536ab78e25333e0d1e39c0fbf37978ff2f0f

[![](https://www.lucidchart.com/publicSegments/view/63a13c5a-eca8-4446-8978-30f0e77e74fc/image.png)](https://www.lucidchart.com/publicSegments/view/63a13c5a-eca8-4446-8978-30f0e77e74fc/image.png)

## 子密钥的衍生方法

HD 钱包中的每个密钥（私钥和公钥）都有 232232 个子密钥。

每个子密钥都用一个序号标识，表示它是这个父密钥衍生出的第几个子密钥，序号从 0 开始计数。

* 序号在 [0,231−10,231−1] 范围的衍生，称为 ** 常规衍生 **（Normal Derivation）
* 序号在 [231,232−1231,232−1] 范围的衍生，称为 ** 硬化衍生 **（Hardened Derivation）、** 增强衍生 ** 或 ** 强化衍生 **

[子密钥衍生](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#child-key-derivation-ckd-functions)（CKD，Child Key Derivation）算法由 [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 定义。

一般的，在衍生子密钥时，将 ** 父密钥 **、** 序号 ** 和 ** 父链码 ** 作为 CKD 的输入，输出一个 256 位的子密钥和一个 256 位的子链码。

这个子链码会在这个子密钥衍生子密钥时，作为 CKD 的输入。

[![](https://www.lucidchart.com/publicSegments/view/f2221b6a-db03-4b0d-886f-c3d13bcff6ac/image.png)](https://www.lucidchart.com/publicSegments/view/f2221b6a-db03-4b0d-886f-c3d13bcff6ac/image.png)

写个程序算一下，主私钥 `116c2daffad72d24cd3c122a65f937ec2743f98952e174ae158bf6ea70c78954` 的第 0 个和第 1 个子密钥。
```
    package main

    import (
        "encoding/hex"
        "fmt"
        "github.com/tyler-smith/go-bip32"
        "github.com/tyler-smith/go-bip39"
    )

    func main() {
        mnemonic := "furnace tunnel buyer merry feature stamp brown client fine stomach company blossom"
        seed := bip39.NewSeed(mnemonic, "")
        masterKey, _ := bip32.NewMasterKey(seed)

        fmt.Println("Master private key\t" + hex.EncodeToString(masterKey.Key))
        fmt.Println("Master public key\t" + hex.EncodeToString(masterKey.PublicKey().Key))
        fmt.Println("Master chain code\t" + hex.EncodeToString(masterKey.ChainCode))

        InspectChildKey(masterKey, 0)
        InspectChildKey(masterKey, 1)
    }

    func InspectChildKey(parentKey *bip32.Key, index uint32) {
        childKey, _ := parentKey.NewChildKey(index)
        fmt.Println(fmt.Sprintf("Child %d private key\t%s", index, hex.EncodeToString(childKey.Key)))
        fmt.Println(fmt.Sprintf("Child %d public key\t%s", index, hex.EncodeToString(childKey.PublicKey().Key)))
        fmt.Println(fmt.Sprintf("Child %d chain code\t%s", index, hex.EncodeToString(childKey.ChainCode)))
    }

    // 输出
    // Master private key   116c2daffad72d24cd3c122a65f937ec2743f98952e174ae158bf6ea70c78954
    // Master public key    02c8022cf8de6472f50f08b8b7e364536ab78e25333e0d1e39c0fbf37978ff2f0f
    // Master chain code    a74b75701aba81dd29e94226696cc0e67d6a5f29398d151c05c09c416dbf0865
    // Child 0 private key  104eedfeeaa8b2c1217c721f375ece3e6981501c7ccd17fcb58867816d01c1b9
    // Child 0 public key   0272fed87974babeee6d01b918d87dcd16d5eabc7eab43c66a547ffea47229563a
    // Child 0 chain code   e2b3998b1df51120f87da3eee1ab6e8b2afb82234d4d6ae62d6e50570e72f737
    // Child 1 private key  7d008006853eea7982e25b7ea325a049161ffa3017c5b80095eda8bc1a2ffb98
    // Child 1 public key   02473bb5ace2f2e2dce4bf7cb3b89ae329c85612753bcb02db5e5d70d95e86e776
    // Child 1 chain code   e2061c5a048003ab0c2060761d1452befb2cba0df9f6f4dc17a8db3ec09114b4
```
根据 [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 的定义，

* 可以从私钥和链码，衍生出其 ** 所有的 ** 子私钥及对应的子公钥（及之后每层所有的子私钥及对应的子公钥）
* 可以从公钥和链码，衍生出其 ** 常规衍生 ** 的子公钥（及之后每层常规衍生的子公钥）
* 无法从某个密钥（公钥和私钥）计算出其父密钥，或同层的其他兄弟密钥

## 扩展密钥

衍生子密钥时需要将密钥、链码和子密钥序号作为 CKD 的输入，三者缺一不可。

为了方便转录，可以将 ** 密钥 ** 和 ** 链码 ** 编码在一起，得到 ** 扩展密钥 **（Extended Key）。

扩展密钥使用 Base58Check 编码，并添加特定的版本前缀。

类型版本前缀的值（十六进制）Base58Check 之后的前缀扩展私钥 0488ade4xprv 扩展公钥 0488b21expub

下面的程序计算例子中的主密钥、主密钥的第 0 个子密钥和主密钥的第 1 个子密钥这三者的扩展密钥。
```
    package main

    import (
        "fmt"
        "github.com/tyler-smith/go-bip32"
        "github.com/tyler-smith/go-bip39"
    )

    func main() {
        mnemonic := "furnace tunnel buyer merry feature stamp brown client fine stomach company blossom"
        seed := bip39.NewSeed(mnemonic, "")

        masterKey, _ := bip32.NewMasterKey(seed)
        firstChildKey, _ := masterKey.NewChildKey(0)
        secondChildKey, _ := masterKey.NewChildKey(1)

        fmt.Println("Master Extended private key\t", masterKey)
        fmt.Println("Master Extended public key\t", masterKey.PublicKey())

        fmt.Println("Child 0 Extended private key\t", firstChildKey)
        fmt.Println("Child 0 Extended public key\t", firstChildKey.PublicKey())

        fmt.Println("Child 1 Extended private key\t", secondChildKey)
        fmt.Println("Child 1 Extended public key\t", secondChildKey.PublicKey())
    }

    // 输出
    // Master Extended private key   xprv9s21ZrQH143K3ixinZag69usQ2CMqDbEkm74p3PWY2ecvUkaiwPGbykMNLfAEwakjwbexs6kKrCDQCGp5vV4yJziz6XB47smbBCmsYhP85Z
    // Master Extended public key    xpub661MyMwAqRbcGD3Btb7gTHrbx42rEgK67z2fcRo86NBboH5jGUhX9n4qDd5LbW56NE3NVxupasmRwZnzN9cNvkvWiG4ZFjY8HqGi1bPXuUR
    // Child 0 Extended private key  xprv9vcPiWn5V1vJdpNtY5qBb5a9xZd8PxT9beX8fYS4eVxm7vrw52N49uwUqHz5vEQjt28o5MbRT4VZaasJQrBUEWxjGhmcb4LVktACFoJ8DyS
    // Child 0 Extended public key   xpub69bk82JyKPUbrJTMe7NBxDWtWbTcoRAzxsSjTvqgCqVjzjC5cZgJhiFxgZmKC676T9tuYAvCTJJq73i114XkMTnJ7o14yfx7tbQ6GVf7D4a
    // Child 1 Extended private key  xprv9vcPiWn5V1vJgMpAdK5KHc9BZYvwfwEJnmU9PnjxCfpTWoNVsg1PPM1rsnseesNdCCoiH3ZW3BVZLuEqskmNnt4jEqhZ79EerwzPR3LvXQo
    // Child 1 Extended public key   xpub69bk82JyKPUbtqtdjLcKek5v7amS5PxA9zPkCB9Zm1MSPbheRDKdw9LLj3VjeRyCkm8gtfzhzmD6sotgKkD5Dn3KhK1NyYEHACc2cSqrsTb
```
扩展密钥使用方便，但要注意：

* 虽然泄露某个扩展公钥不会丢币，但会导致以此为根节点衍生出的扩展公钥全部泄露，破坏了隐私性
* 泄露扩展公钥和该公钥衍生出的之后任一代公钥对应的私钥，有被推导出该扩展公钥所有后代私钥的可能

基于多一层安全的考虑，[BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 定义了两种子密钥衍生方案。

* 常规衍生，从扩展公钥只能衍生出前 231231 个子公钥
* 硬化衍生，后 231231 个子公钥只能从扩展私钥衍生（从扩展私钥能衍生其之后的所有子密钥）

## 衍生路径

为了能方便表示密钥间关系，定义了衍生路径（Derivation Path）的概念。

* 序号之间以 `/` 分隔
* `m` 表示主密钥
* `i` 表示第 ii 个常规衍生的子密钥，即第 ii 个子密钥
* `i'` 表示第 ii 个硬化衍生的子密钥，即第 (231+i)(231+i) 个子密钥

`m/0'/1'/2` 表示主密钥的第 0 个强化衍生子密钥的第 1 个强化衍生子密钥的第 2 个常规衍生子密钥（树形结构）。

扩展密钥加上衍生路径，可以确定 HD 钱包里的一个密钥及从这个密钥衍生的之后所有层的子密钥（以这个密钥为根的子树）。

HD 钱包里的密钥是树形结构，可以无限层衍生下去，为了能让不同钱包之间相互兼容，[BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) 对衍生路径提出了一个规范建议。

    m/purpose'/coin_type'/account'/change/address_index

* `purpose` 总是设为 44，代表钱包遵循 [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) 规范
* `coin_type` 代表币种（[对应关系](https://github.com/satoshilabs/slips/blob/master/slip-0044.md)），Bitcoin（BTC）用 0 ，Bitcoin Cash（BCH）用 145，Bitcoin SV（BSV）用 236
* `account` 代表逻辑上的钱包 “账户”，从 0 开始计数
* `change` 代表地址类型，为 0 表示是收款地址，为 1 表示是找零地址
* `address_index` 是地址索引，从 0 开始计数，表示是第几个地址

[![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/OXLqNXK.png)](https://aaron67-public.oss-cn-beijing.aliyuncs.com/OXLqNXK.png)

我初始化了一个 HD 钱包，使用衍生路径 `m/44'/236'/0'` 作为存放 Bitcoin SV（BSV）的 “账户”，那么，

* 我的第一个收款地址是公钥 `m/44'/236'/0'/0/0` 对应的地址，第二个收款地址是公钥 `m/44'/236'/0'/0/1` 对应的地址，以此类推
* 当我完成第一次支付并存在找零时，会找零到地址 `m/44'/236'/0'/1/0`，下一次支付找零到的地址会是 `m/44'/236'/0'/1/1`，以此类推
* 如果想再新建一个 BSV “账户” 另作他用，可以使用路径 `m/44'/236'/1'`
* 如果还想用这个 HD 钱包存放 BCH，可以使用路径 `m/44'/145'/0'`

注意，[BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) 不是强制标准，你可以随意使用任何衍生路径，只要在备份 HD 钱包的时候 ** 务必记住这个路径 ** 就好。

在做一些有关 HD 钱包细节的计算时，<https://iancoleman.io/bip39/> 这个工具非常不错，推荐给你。

## HD 钱包的优势

HD 钱包在备份时十分方便。

* 只需要备份 ** 助记词 ** 和 ** 密语 **，就等于备份了整个钱包内的所有私钥
* 除此之外，你还要记下使用的 ** 衍生路径 **，这样才能知道使用了哪些私钥

另外，从扩展公钥可以常规衍生子公钥及对应地址而不用访问扩展私钥或私钥本身，这是 HD 钱包一个很重要的安全特性。

密钥间的树形结构，与机构的部门设置十分相似，如果一家企业准备使用比特币进行财务收支，可以：

* 将路径 `m/0'/0'/x'` 的扩展公钥交给各销售部门独自管理和使用
  * 销售部门可以为每笔订单生成不同的收款地址，方便状态跟踪
  * 因为从扩展公钥无法衍生出子私钥，所以销售部门只能收款而无法支付账户里的比特币
* 将路径 `m/0'/0'` 的扩展公钥交给市场部，市场部可以查阅所有订单的销售记录，同样无法支付比特币
* 将路径 `m/0'/0'` 的扩展私钥交给财务部，财务部可以用这个更上层的扩展私钥，管理整个公司的加密资产

配合 [BIP-45](https://github.com/bitcoin/bips/blob/master/bip-0045.mediawiki) 定义的 HD 钱包多签方案，可以方便、安全、灵活的管理公司的加密资产。

# WIF

还记得 [《比特币的私钥和公钥》](https://aaron67.cc/2018/12/23/bitcoin-keys/) 文章最后留下的问题吗？

> WIF 压缩和不压缩格式表示的私钥，其结果从长度上看并没有明显的区别，为什么要这么做

我们常说，一个公钥会对应一个确定的地址，因为地址是公钥哈希的编码。但比特币公钥可以用不压缩格式和压缩格式两种方法表示，这样可以计算出两个不同的公钥哈希，对应到两个地址。

对于私钥 `98fd2a819a382f8e142e38242f6caf2a2f6f58e7fe6ca5f23c5b0818b15b4ba6`，对应的公钥为

    x = da52d817a5ae3555f36a94528322eb47016f1334b798f5b4fa614a892dabb3ea
    y = f314bf38816673c55c1708cf1e36b55c936db97618d6460c4d223be83bec7788

如果用不压缩格式表示公钥，公钥为

    04 da52d817a5ae3555f36a94528322eb47016f1334b798f5b4fa614a892dabb3ea f314bf38816673c55c1708cf1e36b55c936db97618d6460c4d223be83bec7788

对应的 P2PKH 地址为 `1B4nPuT41LBxzumQqhrPDsd4NF7DZSWgyQ`。

如果用压缩格式表示公钥，公钥为

    02 da52d817a5ae3555f36a94528322eb47016f1334b798f5b4fa614a892dabb3ea

对应的 P2PKH 地址为 `1BJhat1AMGYbT9HYJxVekoCaPaqB9ZyTyF`。

对于早期的钱包软件，都是直接使用不压缩格式的公钥，计算对应的地址用于收款。

后来，人们发现公钥可以用压缩的格式存储，这样可以节省约一半的存储和传输空间，钱包开始逐渐使用压缩格式的公钥。

如果一个钱包软件支持两种格式的公钥，当你向钱包里导入私钥时，钱包就懵逼了，由于不知道你原来使用的是什么格式的公钥，所以需要在区块链里搜索这两个地址上锁定的 UTXO 从而计算出正确的 “账户余额”，这会带来混乱。

为了向后兼容，定义了 WIF 压缩格式。

* 在实现了压缩格式公钥的较新的钱包中，私钥只能且永远被导出为 WIF 压缩格式（以 `K` 或 `L` 为前缀）
* 对于较老的没有实现压缩格式公钥的钱包，私钥只能被导出为 WIF 不压缩格式（以 `5` 为前缀）

这样做的目的就是为了给导入这些私钥的钱包一个信号：钱包需要使用什么格式的公钥来计算地址，搜索区块链。

# 总结

使用 “钱包” 软件，能方便的收发比特币。

早期的钱包都是离散私钥钱包，包含在全节点软件中。

* 离散钱包中的私钥之间彼此毫无关联，备份和使用时有诸多不便
* 全节点软件会同步所有的区块数据，这是一个十分耗时的操作，对钱包来说，也并不需要全部的区块数据

确定性钱包从一个种子衍生出钱包内的所有私钥，分层确定性钱包主要由 [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)、[BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) 和 [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) 共同定义。

* [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)，为了避免管理一堆私钥的麻烦，定义钱包内密钥的树形结构及分层衍生方案
* [BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)，定义助记词，让备份种子更方便
* [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)，对 [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 的衍生路径提出规范建议

HD 钱包的幕后细节涉及到很多内容，希望这篇文章能帮你理解它们。

* 助记词（Mnemonic）
* 从随机序列（熵）创建助记词
* 种子（Seed）
* 主密钥（Master Key）
* 从助记词创建种子（PBKDF2），从种子衍生主密钥（HMAC-SHA512），子密钥的衍生方法（CKD）
* 扩展密钥（Extended Key）
* 常规衍生（Normal Derivation）和硬化衍生（Hardened Derivation）
* 衍生路径（Derivation Path）
* HD 钱包的备份

# 参考

* 精通比特币（第二版）[译文](https://wizardforcel.gitbooks.io/masterbitcoin2cn/content/) [原文](https://github.com/bitcoinbook/bitcoinbook/)
* [PBKDF2 算法概述](https://www.voidcn.com/article/p-vdtfkabe-nq.html)
* [理解开发 HD 钱包涉及的 BIP32、BIP44、BIP39](https://learnblockchain.cn/2018/09/28/hdwallet/)
* [数字货币钱包 - 助记词 及 HD 钱包密钥原理](https://zhuanlan.zhihu.com/p/34184347)
* [ELI5: What’s the difference between a child-key and a hardened child-key in BIP32](https://bitcoin.stackexchange.com/questions/37488/eli5-whats-the-difference-between-a-child-key-and-a-hardened-child-key-in-bip3)
* [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)
* [BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
* [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)
* [tyler-smith/go-bip39](https://github.com/tyler-smith/go-bip39)
