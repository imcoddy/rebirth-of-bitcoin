# 比特币安全

本章主要介绍比特币安全相关的信息，包括冷钱包，热钱包以及硬件钱包的例子。

收发比特币一般都直接使用钱包软件。在了解了比特币系统和钱包的工作细节后，这篇文章将介绍：

- 选择和使用钱包软件时需要注意什么
- 如何安全的生成助记词
- 使用 ** 热钱包 ** 在日常生活中收发比特币
- 使用 ** 冷钱包 ** 和 ** 观察钱包 ** 来满足更强的安全性需求
- 硬件钱包

掌握这些内容，能让你在安全存储私钥的同时也可以方便的使用比特币。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/w95gdbU.jpg)

<!-- more -->

# First Things First

选择什么钱包软件，如何使用钱包，都是在考虑两个核心问题：

- 私钥的安全性如何
- 是不是方便易用。软件是否实现了我需要的功能，收发比特币时是不是易于操作有不错的用户体验

比特币钱包的种类很多，有运行在电脑桌面上的软件钱包，有安装在手机上的钱包 App，还有需要配合特定硬件使用的硬件钱包。

按按私钥是否联网，可以分为：

- 冷钱包，钱包中的私钥在 ** 任何时候 ** 都不曾接触网络
- 热钱包，私钥直接接触网络

在比特币系统中，私钥意味着一切。HD 钱包中的私钥都从种子（Seed）计算而来，种子从助记词（Mnemonic）和密语（Passphrase）计算而来，如果你指定了密语，在备份 HD 钱包时需要同时记录这两者。为了避免计算机和手机病毒窃取信息，** 你不应该使用任何电子设备记录助记词 **，一般都采用纸笔抄录的方式备份，或记录在更坚固的介质上。同时直接将密语记在脑子里，作为保证钱包安全的第二因素，在助记词意外泄露后避免资金损失。

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/CsI3Kod.png)</div>

钱包安全涉及到三个部分：

1. 安全的生成助记词
2. 保证助记词备份介质的安全
3. 在使用钱包软件时保护私钥

对于 [2] 如何保证备份介质的安全，这里只提供思路而不展开描述，套路很多，如何选择看你自己。你可以将助记词抄在一张纸上，存放在你认为足够安全的地方，这种方式虽然简单但无法抵御单点失败，一旦这个地方不再安全便有泄露助记词的风险。一般的，可以将助记词抄写到三张纸上，分开存放在不同的地方，每张纸都只记录一部分内容，至少需要两张纸才能知道所有的助记词。

在接收比特币时，需要提供收款地址。HD 钱包在生成地址时，根据路径衍生方式的不同，需要访问扩展私钥或扩展公钥。

在支付比特币时，需要依次完成创建交易、签名交易和广播交易三个 ** 相互独立 ** 的步骤。

符合 [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)、[BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) 和 [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) 标准的 HD 钱包，会使用形如 `m/44'/236'/0'` 的衍生路径，使用路径 `m/44'/236'/0'/0/x` 作为收款地址，使用路径 `m/44'/236'/0'/1/x` 作为找零地址，所以对 HD 钱包而言：

- 只需要访问扩展公钥（xpub），就可以生成地址（收款地址和找零地址）
- 需要访问扩展私钥（xprv），才可以对交易签名

场景 | 需要网络？ | 需要访问私钥？ | 说明
--- | --- | --- | ---
生成地址 | ✖ | ✖ |
创建交易 | ✔ | ✖ | UTXO 需要从网络同步
签名交易 | ✖ | ✔ |
广播交易 | ✔ | ✖ |

显而易见，从头初始化一个 HD 钱包，或从助记词、密语和衍生路径恢复 HD 钱包，或直接导入扩展私钥（xprv）恢复 HD 钱包，才可以支付比特币。直接导入扩展公钥（xpub）恢复 HD 钱包，意味着你只能生成地址接收而无法签名交易支付比特币，这样的钱包称为 ** 观察钱包 **，观察钱包中没有私钥。

你可以在 ** 一直离线 ** 的电脑或手机上生成冷钱包，只用来对交易签名，这台设备一直处于离线状态，所以你不用特别担心私钥通过网络泄露。联网的观察钱包可以实时从网络同步 UTXO 数据，更新钱包 “余额”，你可以在观察钱包中生成地址接收比特币，组合 UTXO 创建未签名的交易。

在需要支付比特币时：

1. 用观察钱包生成一笔未签名的交易，将交易数据保存为文件或二维码
2. 通过 U 盘或扫描二维码转移未签名的交易数据到离线设备（二维码方式的安全性更高）
3. 用冷钱包对交易签名
4. 通过 U 盘或扫描二维码转移签名后的交易数据到联网设备
5. 广播交易

整个发送过程冷钱包一直处于离线状态，私钥也没有接触网络。

一个要注意的点是，在保障钱包安全性的同时，不应完全忽略使用体验。在对安全性要求不那么高的场景，例如用于日常支付的零钱钱包，为了方便完全可以直接使用你信赖的热钱包软件而不用过于担心，就像你不太担心丢失钱包而损失银行存款一样。对于大额比特币的 “存储”，可以考虑使用冷钱包方案，牺牲一些使用体验换来更高的安全保障。

另一个要注意的点是，你应该只从软件官方网站和手机官方应用商店这样的可信渠道下载钱包。如果可以的话，记得验证安装包的哈希和 GPG 签名。

下面将详细演示 Bitcoin SV（BSV）钱包的使用方法，对于 Bitcoin（BTC）和 Bitcoin Cash（BCH）钱包，原理都是相同的。

# 热钱包

由于热钱包的私钥直接接触网络，所以安全性上会稍显不足，但热钱包简单方便，将其作为日常使用的零钱钱包，再合适不过。你可以根据自己的喜好，在官网的 [钱包推荐列表](https://bitcoinsv.io/services/wallets-and-exchanges/) 中挑选。

<div style="width: 65%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/xlK6Seo.png)</div>

我常用的钱包有：

- [打点钱包](https://www.ddpurse.com/)，轻量易用的手机钱包，无需下载 App，关注官方微信公众号便可直接使用
- [Simply Cash](https://simply.cash/)，简洁优雅的手机钱包，支持冷钱包功能
- [MoneyButton](https://moneybutton.com/)，操作体验优异的在线钱包
- [IFWallet](https://www.ifwallet.com/)，简单好用的 HD + 托管 “双核” 手机钱包
- [ElectrumSV](https://electrumsv.io/)，功能强大的桌面钱包，支持冷钱包功能，兼容主流的硬件钱包

以打点钱包为例。

- 三个功能已足够日常使用
<div style="width: 20%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/NYDKYYV.png)</div>

- 可以支付 BSV 到地址或微信好友，也可以发送 BSV 红包
<div style="width: 20%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/JKmRs3a.png)</div>

- 记得备份助记词
<div style="width: 20%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/4rJACZr.png)</div>

你可以通过下面的视频了解打点钱包的更多细节。视频由热心网友 “简美小视频” 制作，非常不错，分享给你。

<video src="https://aaron67-public.oss-cn-beijing.aliyuncs.com/ddpurse-tutorial.mp4" controls="controls" style="max-width: 100%; display: block; margin-left: auto; margin-right: auto;"></video>

# 冷钱包

在 “存储” 大额比特币时需要处处小心谨慎，避免私钥泄露。

## 安全的生成助记词

如果你担心钱包软件在生成助记词时作恶，可以选择自己生成，通过导入助记词的方式在离线设备上恢复冷钱包。

https://iancoleman.io/bip39/ 是一个颇受欢迎的工具，源码开源在 [GitHub](https://github.com/iancoleman/bip39) 上。你可以将项目代码下载下来，拷贝到 ** 离线电脑 ** 上运行，避免信息泄露。选择要生成的助记词个数，每点一下 “GENERATE” 按钮就可以生成一组新的助记词。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/2IdSSMC.png)

助记词是从熵计算出来的，所以你也可以通过抛硬币或掷骰子的方式生成一个足够随机的熵，从而计算助记词。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/Pc50y7N.png)

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/AT2AHuT.png)

你甚至能通过抛硬币或掷骰子的方式直接生成助记词，但因为 [BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) 助记词的最后一个单词包含校验和，所以过程要麻烦的多，我写了一个 [小工具](https://github.com/gitzhou/mnemonic-last-word) 来解决这个问题，你可以 ** 离线运行 ** 它，原理在助记词一节中详细介绍过，这里只简单说一下。

[BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) 助记词的最后一个单词其实由两部分确定：

- 熵的低 $M$ 位，$M$ 取决于熵的长度
- 熵 SHA256 结果的高 $N$ 位，$N$ 取决于熵的长度

对应关系为：

熵的长度（二进制位） | 助记词的个数 | M（二进制位个数） | N（二进制位个数）
--- | --- | --- | ---
128 | 12 | 7 | 4
160 | 15 | 6 | 5
192 | 18 | 5 | 6
224 | 21 | 4 | 7
256 | 24 | 3 | 8

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/e9wIAzF.png)</div>

假如我想用自己的方法选取 24 个助记词：

1. 选择助记词语言
2. 使用自己认可的方法先确定前 23 个助记词，填到 [2] 里
3. 抛硬币或用其他方法得到一个 3 位的 ** 随机 ** 二进制串，填到 [3] 里
4. 点 “Calculate” 按钮计算最后一个助记词

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/aLiSkp9.png)</div>

同样的，自己生成助记词的套路有很多，如何选择看你自己，不论使用何种方法，都需要保证：

- 整个生成过程都处于离线环境
- 选取方法 ** 足够随机 **

## 如何使用冷钱包

为了方便截图演示，这里事先用工具生成了一组助记词，指定的密语是 `satoshi`，使用的衍生路径是 `m/44'/236'/0'`。

```
cry devote two glare orchard below box fatigue box document jar night
```

这个 HD 钱包的扩展公钥（xpub）是：

```
xpub6CbEfGaUsev7P1pDhNKm1YsL9xRNHTDPsS4u9AHsdNnHwvokk6ULWUByqdULY5SH889Bqdknkn6erGCyQreRLns2vrLnbfWSZM3z682jijX
```

** 请务必注意！对于真实使用的 HD 钱包，暴露这些信息将会带来灾难性的后果 **！

### ElectrumSV

在 ** 离线电脑 ** 上恢复冷钱包，菜单 `File --> New/Restore`。最后一步指定的密码会被用于加密本地的钱包文件，只对当前钱包文件有效。

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/Eb1hwwS.png)</div>

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/1fDJqLL.png)</div>

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/nZ4WL7i.png)</div>

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/BA9bYTK.png)</div>

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/AGqUAfn.png)</div>

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/KW4LdQx.png)</div>

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/2FdQ1DB.png)</div>

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/DaYHR2r.png)</div>

恢复好冷钱包后，对比一下钱包生成的地址和工具生成的是否一致。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/MFypvvy.png)

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/tEXshOM.png)

从菜单 `Wallet --> Information`，查看钱包的扩展公钥。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/hKxropK.png)

在 ** 联网电脑 ** 上导入扩展公钥恢复观察钱包，菜单 `File --> New/Restore`。

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/bRWXpeF.png)</div>

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/FIqmcAI.png)</div>

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/IXo16ge.png)</div>

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/mexAtNv.png)

从提示可以看出，观察钱包不能支付。在恢复完成后，对比一下观察钱包生成的地址和冷钱包生成的是否一致。

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/vxdnLST.png)</div>

至此，钱包恢复完毕。

接收 BSV 时，只需使用观察钱包就可以生成收款地址，虽然观察钱包联网，但因其不包含私钥所以不会有致命风险。

支付 BSV 时：

1. 用观察钱包生成一笔未签名的交易

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/kIMNFxk.png)</div>

2. 将未签名的交易数据保存为文件或二维码。你能看到，观察钱包无法签名交易，未签名的交易也无法广播

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/PDc2zGa.png)</div>

未签名的交易数据文件，内容如下

```
{
    "hex": "0100000001f2d53311d7299e2be6db46c1d2f6543bac00f067bb643d694ed0dee9ec6f8d30000000005701ff4c53ff0488b21e037fc3c84080000000a8d961a0e0af5920601522cad94590c447d739bf63ec8f2ec752743c953c8fe2021d1c7fbb4b663975ef24ac1e6c6fa18880168f930c8e86a19929d33e478e56e700000100feffffff30750000000000000170740000000000001976a9148fdb3f6131ad653d40de981ab7eda7d2dd71995b88acdab20800",
    "complete": false,
    "final": true
}
```

3. 在冷钱包上，从文件加载交易，菜单 `Tools --> Load transaction --> From file`

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/nuKBBHp.png)</div>

4. 用冷钱包对交易签名

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/JoAy8DA.png)</div>

5. 将签名后的交易数据保存为文件或二维码

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/SgnhKAc.png)</div>

签名后的交易数据文件，内容如下

```
{
    "hex": "0100000001f2d53311d7299e2be6db46c1d2f6543bac00f067bb643d694ed0dee9ec6f8d30000000006a4730440220636e8c8ad021d40d1089a5caa54e4ce9c83854f04018e4a1397516676d95909002203fb9d1d71b08744b2973f0af53f8fdbde1a549f33b7b22252a0c6276fe7ddeda412103b7d4f78bcf09c86cde4e2bcf1a8f31d5a0498105a53df3517c5f64002a87b9c8feffffff0170740000000000001976a9148fdb3f6131ad653d40de981ab7eda7d2dd71995b88acdab20800",
    "complete": true,
    "final": true
}
```

6. 在观察钱包上加载签名后的交易文件并广播

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/uCkvSEk.png)</div>

注意，广播交易时除了直接使用钱包软件，还可以借助第三方服务或 API。

  - [Blockchair](https://blockchair.com/broadcast)
  - [WhatsOnChain.com](https://whatsonchain.com/broadcast)
  - [BitIndex API](https://www.bitindex.network/docs.html)

至此，全网都会收到 [这笔交易](https://whatsonchain.com/tx/4eba284fcf5653393870f3d265abbf7f65e9b3e272e7dd451872d772f94a7719)，支付完成。

必须要指出的是，使用 ElectrumSV 生成的助记词（`New -> Standard wallet -> Create a new seed`），是 ** 不符合 ** [BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) 规范的，请务必注意。

<div style="width: 60%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/qLhzYhg.png)</div>

<div style="width: 60%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/bo392hV.png)</div>

### Simply Cash

使用 Simply Cash 作为冷钱包，过程也是类似的。

在 ** 离线手机 ** 上恢复冷钱包并核对地址。

<div style="width: 20%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/kN8MSeC.png)</div>

<div style="width: 20%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/QjZe6v6.png)</div>

在 ** 联网手机 ** 上恢复观察钱包并核对地址。

<div style="width: 20%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/wXxwjVw.png)</div>

<div style="width: 20%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/wEuifgr.png)</div>

支付 BSV 时：

1. 在观察钱包中生成一笔未签名的交易

<div style="width: 20%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/Bn6Jv3N.png)</div>

这个二维码解析之后的内容为

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/1zkhtKL.png)</div>

2. 用冷钱包扫描二维码加载未签名的交易并签名

<div style="width: 20%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/CgDC0Zs.png)</div>

<div style="width: 20%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/wPapYWD.png)</div>

这个二维码解析之后的内容为

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/87rEQnK.png)</div>

3. 用观察钱包扫描二维码加载签名后的交易并广播

<div style="width: 20%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/LaMXyOY.png)</div>

至此，全网都会收到 [这笔交易](https://whatsonchain.com/tx/08793a39d3286a40662deae2bc5e8e8de598079268a3c27eeda64b858ded91d9)，支付完成。

你可以通过下面两个视频了解 Simply Cash 的更多细节。视频由热心网友 “简美小视频” 制作，非常不错，分享给你。

<video src="https://aaron67-public.oss-cn-beijing.aliyuncs.com/simply-cash-tutorial-0551.mp4" controls="controls" style="max-width: 100%; display: block; margin-left: auto; margin-right: auto; max-height: 650px;"></video>

<p></p>

<video src="https://aaron67-public.oss-cn-beijing.aliyuncs.com/simply-cash-tutorial-0207.mp4" controls="controls" style="max-width: 100%; display: block; margin-left: auto; margin-right: auto; max-height: 650px;"></video>


# 硬件钱包

热钱包足够方便但存在一定的安全隐患，适合 “存放” 小额零钱用于日常支付。冷钱包足够安全但支付时多有不便，适合 “存放” 大额且不经常使用的比特币。

知名的硬件钱包厂商有 [Ledger](https://www.ledger.com/)、[TREZOR](https://trezor.io/) 和 [Keepkey](https://www.keepkey.com/)，借助精心设计的软硬件，硬件钱包能在方便使用的同时兼顾较高的安全性。网上介绍和比较硬件钱包的文章非常多，这里不再赘述，你可以从 [这篇文章](https://www.buybitcoinworldwide.com/zh-cn/bitcoin-wallets/) 开始了解。

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/eNYoQxL.jpg)</div>

硬件钱包的安全性一直以来都备受关注，虽然厂商会采用各种方式降低产品被攻破的可能（如安全芯片、防暴拆、开源等），但事情没有绝对。硬件钱包从设计生产到装配、运输、交货会涉及更多环节，选择信任什么同样看你自己。

# 一个地址只用一次

为了保证安全性和保护个人隐私，一个比特币地址只应被使用一次。

当郭达和蔡明都要向你支付比特币时，你应向他们提供从未使用过的不同的收款地址。

在支付比特币时，大多数 HD 钱包也都会使用找零地址来满足这样的需求，以 ElectrumSV 钱包为例，默认设置会勾选 “使用找零地址” 的选项。

<div style="width: 50%; margin: auto">![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/G14MtjW.png)</div>

当郭达付款给蔡明并存在交易找零时，让我们看看不同的设置会带来怎样的效果。为了方便识别，蔡明的收款地址用红线标注。

- 不使用找零地址

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/yHjNmoH.png)

- 使用找零地址

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/k4UnEID.png)

- 使用多个找零地址

这是 ElectrumSV 提供的一个特性，使用多个找零地址 “打散” 面值较大的单个找零。

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/pY83GVe.png)

![Imgur](https://aaron67-public.oss-cn-beijing.aliyuncs.com/XlsSDu3.png)

通过上面三笔交易你能清楚的看到，不使用找零地址能直接确定蔡明的收款地址，而使用找零地址会让交易活动变得难以追踪。

# 总结

了解和掌握如何安全的收发比特币十分重要，这关系到资金安全。

钱包的安全性受多重因素共同影响，其高低取决于解决方案中最脆弱的部分。深入了解比特币协议的细节和系统的运作方式，也有助于你设计出更适合自己的钱包方案，例如你可以在收发比特币时结合多重签名技术来提升安全性。

让钱包一直处于离线环境从不接触过网络能有效避免私钥泄露，但这会给支付带来一些麻烦。相较于一概而论，针对不同的使用场景选择不同的钱包方案，找到安全性和易用性的平衡点才是更明智的选择。

钱包方案 | 比特币额度 | 支付频率 | 易用性 | 安全性
--- | --- | --- | --- | ---
热钱包 | 小 | 高 | 高 | 一般
硬件钱包 | 大 | 中 | 高 | 高
冷钱包 | 大 | 低 | 一般 | 极高
