# 钱包的备份和恢复


当你导入助记词（Mnemonic）恢复 HD 钱包时，需要同时指定衍生路径（Derivation Path）和用户密语（Passphrase）。

由于历史原因，比特币经历了 BTC 至 BCH 以及现在 BSV 的名称变更，其 HD 钱包的路径也有所改变，这篇文章汇总了一些常用 BSV 钱包默认使用的衍生路径信息。

钱包 | 衍生路径 | 用户密语 | 备注
--- | --- | --- | ---
[打点钱包](https://www.ddpurse.com/) | m/44'/0'/0' | 无 |
[Simply Cash](https://simply.cash/) | 默认是 m/44'/145'/0' | 无 | 新建钱包时可以自定义衍生路径
[Money Button](https://www.moneybutton.com/) | m/44'/0'/0' | 无 |
[HandCash](https://handcash.io/) | m/0' | 无 |
[RelayX](https://relayx.io/) | m/44'/236'/0' | 无 |
[Yours.org](https://www.yours.org/) | m/44'/0'/0' | 无 | 参考 [How To Import Your Yours Wallet Into Electrum SV](https://www.yours.org/content/how-to-import-your-yours-wallet-into-electrum-sv-c811b3dea0fb)
[Centbee](https://www.centbee.com/) | m/44'/0 | 用户设定的 PIN 码，如下图 |

<div style="width: 30%; margin: auto">![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522233048.png)</div>

请注意，通过 [ElectrumSV](https://electrumsv.io/) 生成的助记词是不符合 BIP-39 规范的，但 ElectrumSV 支持导入 BIP-39 助记词。

以在 ElectrumSV 中恢复 Centbee 钱包为例，步骤截图如下。

1. 菜单`File -> New/Restore`，通过导入助记词的方式新建一个“标准”钱包

<div style="width: 50%; margin: auto">![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522234834.png)</div>

<div style="width: 50%; margin: auto">![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522233612.png)</div>

<div style="width: 50%; margin: auto">![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522233706.png)</div>

2. 在 [1] 处填写你从 Centbee 中备份的助记词，打开 Options 并勾选“指定用户密语”和“这是 BIP-39 助记词”两项

<div style="width: 50%; margin: auto">![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522234020.png)</div>

3. 设置用户自定义的密语，即填入你在 Centbee 中设置的 PIN 码

<div style="width: 50%; margin: auto">![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522234113.png)</div>

4. 设置衍生路径

<div style="width: 50%; margin: auto">![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522234142.png)</div>

5. 对比一下地址，完成钱包导入

<div style="width: 50%; margin: auto">![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522234411.png)</div>

<div style="width: 30%; margin: auto">![](https://aaron67-public.oss-cn-beijing.aliyuncs.com/20190522235714.png)</div>

## 总结

钱包的路径的历史遗留问题造成了一定的困扰，我们建议钱包应用开发者从此以 `m/44'/0'/0'` 为默认路径。
