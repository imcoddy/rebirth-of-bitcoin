# 钱包的备份和恢复

## 助记词的生成和备份

比特币钱包的备份实质上是能够重新生成相同的一系列私钥。现在的钱包基本支持 BIP39 提案，这意味着绝大多数时候用户只需要备份好十二个 (部分钱包是二十四个) 单词即可。助词词通常在下载并启用比特币钱包应用时随机自动生成，切记将其抄写在纸上并妥善保管。

更高阶的用户可以自行指定衍生路径（Derivation Path）和用户密语（Passphrase），但在恢当你导入助记词（Mnemonic）恢复 HD 钱包时，需要同时指定完全相同的衍生路径和用户密语才能正常恢复。

具体的技术细节请参阅 [HD 钱包](chapter-wallet/hd-wallet.md) 一节。

## 常见衍生路径

由于历史原因，比特币经历了 BTC 至 BCH 以及现在 BSV 的名称变更，其 HD 钱包的路径也有所改变，本节汇总了一些常用 BSV 钱包默认使用的衍生路径信息。

钱包 | 衍生路径 | 用户密语 | 备注
--- | --- | --- | ---
[打点钱包](https://www.ddpurse.com/) | m/44'/0'/0' | 无 |
[Simply Cash](https://simply.cash/) | 默认是 m/44'/145'/0' | 无 | 新建钱包时可以自定义衍生路径
[Money Button](https://www.moneybutton.com/) | m/44'/0'/0' | 无 |
[HandCash](https://handcash.io/) | m/0' | 无 |
[RelayX](https://relayx.io/) | m/44'/236'/0' | 无 |
[Yours.org](https://www.yours.org/) | m/44'/0'/0' | 无 | 参考 [How To Import Your Yours Wallet Into Electrum SV](https://www.yours.org/content/how-to-import-your-yours-wallet-into-electrum-sv-c811b3dea0fb)
[Centbee](https://www.centbee.com/) | m/44'/0 | 用户设定的四位数字 PIN 码 |


## 钱包恢复实践

我们将以在 ElectrumSV 中恢复 Centbee 钱包为例，具体步骤如下。

请注意，通过 [ElectrumSV](https://electrumsv.io/) 生成的助记词是不符合 BIP39 规范的，但 ElectrumSV 支持导入 BIP39 助记词。

1. 菜单 `File -> New/Restore`，通过导入助记词的方式新建一个 “标准” 钱包

![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522234834.png)

![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522233612.png)

![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522233706.png)

2. 在 [1] 处填写你从 Centbee 中备份的助记词，打开 Options 并勾选 “指定用户密语” 和 “这是 BIP39 助记词” 两项

![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522234020.png)

3. 设置用户自定义的密语，即填入你在 Centbee 中设置的 PIN 码

![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522234113.png)

4. 设置衍生路径

![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522234142.png)

5. 对比一下地址，完成钱包导入

![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522234411.png)

![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522235714.png)

## 总结

由于不同的钱包实现的衍生路径不同，容易对用户造成了一定困扰。本书强烈推荐比特币钱包开发者使用 `m/44'/0'/0'` 作为默认衍生路径。
