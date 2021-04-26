# Paymail 协议

比特币旨在用作点对点现金。当白皮书和节点软件发布时，系统创建了两种汇款方式。

> 比特币有两种汇款方式。 如果收件人在线，你可以输入他们的 IP 地址，它将直接连接并获得一个新的公钥并发送带有注释的交易。 如果收件人不在线，那么可以发送到他们的比特币地址，即他们给你的公钥的哈希。 他们在下次上线时获得包含该交易的区块。这种方法的缺点是没有办法发送注释信息，而且如果多次使用同一地址的话，可能会丢失一些隐私性。但如果两个用户都不能同时在线或收件人无法接收连接，它是一个有用的替代方案。--[中本聪](https://nakamotostudies.org/emails/bitcoin-list-bitcoin-v0-1-alpha-release-notes/)

## 概述 - Paymail

Paymail 是现代支付到 IP，是一种将人类可读名称附加到机器可读端点的方法，就像原始比特币一样。

这意味着我们可以拥有看起来像电子邮件地址的名称，这些名称能够发送交易，签名和加密交易，以及执行高级合同。Paymail 还与电子邮件完全兼容。

由于 Paymail 也可用于托管 BIP 270 端点，因此 Paymail 也是现代付费方式的入口点。

最后，使用 Miner ID 加扩展来广播和验证交易，我们可以直接向矿工询问交易是否有效。

因此，使用 Paymail，BIP 270 和 Miner ID，我们可以进行点对点交易，从而使钱包可以放弃运行节点的要求。

钱包不需要扫描区块链来查找他们的交易。他们将直接从发件人接收交易。

这将使每个钱包的运营成本大大降低，同时使一切更加用户友好。

钱包将不再发送到地址。只付钱。

**Paymail** 是比特币 SV 钱包协议的集合，允许在生态系统中的所有钱包中提供一组简化的用户体验。

❌不再使用复杂的 `17Dx2iAnGWPJCdqVvRFr45vL9YvT86TDsn` 地址

✅使用直观的付款处理方式 `<alias>@<domain>.<tld>`

Paymail 协议的目标是：

* 提供对用户友好的可记忆式目标支付格式
* 无需许可的自由集成
* 可以自行托管或委托的服务
* 自动服务检测
* PKI 基础设施
* 跨钱包的一次性构建任何类型交易的输出脚本
* 请求响应式的身份验证
* 安全和策略管理
* 可扩展和被发现的能力

##[bsvalias 协议](https://bsvalias.org/index.html#bsvalias)

相关协议族统称为 `bsvalias` 协议。在撰写本文时，这些包括：

* [BRFC 规格](https://bsvalias.org/01-brfc-specifications.html)
* [服务发现](https://bsvalias.org/02-service-discovery.html)
* [公钥基础设施](https://bsvalias.org/03-public-key-infrastructure.html)
* [付款地址](https://bsvalias.org/04-payment-addressing.html)

##[ Paymail](https://bsvalias.org/index.html#Paymail)

**Paymail** 是执行以下协议的名称：

* [服务发现](https://bsvalias.org/02-service-discovery.html)
* [公钥基础设施](https://bsvalias.org/03-public-key-infrastructure.html)
* [基本地址解析](https://bsvalias.org/04-01-basic-address-resolution.html) 从 [支付解决](https://bsvalias.org/04-payment-addressing.html) 方案组

该 **Paymail** 品牌被保留用于产品和服务，在最低限度，实现每个以上。

##[扩展协议](https://bsvalias.org/index.html#extension-protocols)

根据 [BRFC 规范](https://bsvalias.org/01-brfc-specifications.html) 中的定义，任何人都可以建议扩展 `bsvalias` 和付费邮件协议，并且根据服务发现协议的 [能力发现](https://bsvalias.org/02-03-capability-discovery.html) 部分，实现可以声明对扩展的支持以允许跨钱包过程。[](https://bsvalias.org/02-service-discovery.html)

扩展协议是未包含在上面定义的核心 Paymail 集中的协议的集合，但是它们与 `bsvalias` 协议和 Paymail 实现完全兼容。目前值得注意的例子包括：

* [发件人验证](https://bsvalias.org/04-02-sender-validation.html)
* [接收者批准](https://bsvalias.org/04-03-receiver-approvals.html)
* [PayTo 协议前缀](https://bsvalias.org/04-04-payto-protocol-prefix.html)
* MultiSig 授权
* 阈值签名组秘密设置和消息签名
* 付款渠道
