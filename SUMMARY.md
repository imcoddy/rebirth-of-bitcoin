# Summary

* [比特币重生计划：重返创世纪](README.md)

-----
* [序言](preface.md)

-----
* [比特币之道](chapter-philosophy/README.md)
    * [诚信为本](chapter-philosophy/honesty.md)
    * [路线图](chapter-philosophy/roadmap.md)
    * [链上扩容](chapter-philosophy/onchain-scaling.md)
    * [重返创世纪](chapter-philosophy/back-to-genesis.md)
    * [中本聪愿景](chapter-philosophy/vision-of-satoshi.md)

-----
* [比特币基本概念](chapter-how-bitcoin-works/README.md)
    * [比特币的隐私模型](chapter-how-bitcoin-works/privacy-model.md)
    <!-- * [比特币的架构](chapter-how-bitcoin-works/overview.md) -->
    <!-- * [比特币常见概念](chapter-how-bitcoin-works/basic-concept.md) -->

-----
* [比特币钱包的使用](chapter-wallet/README.md)
    <!-- * [钱包的备份和恢复](chapter-wallet/backup-and-restore.md) -->
    <!-- * [HD 钱包](chapter-wallet/hd-wallet.md) -->
    <!-- * [Paymail 协议](chapter-wallet/paymail.md) -->
    <!-- * [比特币钱包推荐](chapter-wallet/recommendations.md) -->
    * [打点钱包](chapter-wallet/ddpurse.md)
    * [MoneyButton](chapter-wallet/moneybutton.md)
    * [Simply Cash](chapter-wallet/simply-cash.md)

-----
* [比特币交易](chapter-transaction/README.md)
    * [比特币交易的结构](chapter-transaction/transaction-structure.md)
    * [加密算法](chapter-transaction/crypto-in-bitcoin.md)
    * [比特币私钥](chapter-transaction/private-key.md)
    * [比特币公钥](chapter-transaction/public-key.md)
    * [比特币地址](chapter-transaction/address.md)
    * [未花费交易输出](chapter-transaction/utxo.md)
    * [比特币签名](chapter-transaction/signature.md)
        * [P2PKH](chapter-transaction/p2pkh.md)
        * [P2SH](chapter-transaction/p2sh.md)
        * [P2IP](chapter-transaction/p2ip.md)
        * [P2RPH](chapter-transaction/p2rph.md)
    * [交易可塑性](chapter-network/malleability.md)
    * [时间锁](chapter-network/timelocks.md)
    * [OP_RETURN](chapter-network/op_return.md)
    <!-- * [广播交易](chapter-transaction/broadcast-transaction.md) -->

-----
* [比特币区块](chapter-block/README.md)
    * [区块的结构](chapter-block/block-structure.md)
    * [区块确认](chapter-block/block-confirmation.md)
    * [零确认的重要性](chapter-block/importance-of-zero-conf.md)

-----
* [比特币脚本系统](chapter-scripting/README.md)
    * [比特币脚本系统](chapter-scripting/overview.md)
    * [比特币的图灵完备性](chapter-scripting/turing-completeness.md)
    * [比特币的脚本编程](chapter-scripting/programming-in-script.md)
        * [比特币交易模板](chapter-scripting/template.md)
        * [比特币智能合约](chapter-scripting/smart-contract.md)
        * [比特币多重签名](chapter-scripting/multisig.md)
        * [比特币 R-Puzzle 签名](chapter-scripting/r-puzzle.md)
    * [提升操作符上限](chapter-scripting/raising-op-code-limits.md)
    * [移除 P2SH](chapter-scripting/sunsetting-p2sh.md)
    * [隔离见证的风险](chapter-scripting/danger-of-segwit.md)
    * [恢复 OP_RETURN](chapter-scripting/restoring-op_return.md)

-----
* [比特币网络](chapter-network/README.md)
    <!-- * [什么是点对点网络](chapter-network/peer-to-peer.md) -->
    * [双重的比特币网络](chapter-network/overlayed-bitcoin-network.md)
    * [IP2IP](chapter-network/ip2ip.md)
    * [支付通道](chapter-network/payment-channel.md)
    * [SPV](chapter-network/spv.md)

-----
* [比特币挖矿](chapter-mining/README.md)
    * [运行节点](chapter-mining/running-bitcoin-node.md)
    * [工作量证明](chapter-mining/proof-of-work.md)
    <!-- * [MinerID](chapter-mining/miner-id.md) -->
    * [双重哈希](chapter-mining/double-hash.md)
    * [节点的专业化](chapter-mining/professionalize.md)
    * [小世界网络](chapter-mining/small-world-network.md)
    * [竞争的本质](chapter-mining/competition.md)
    * [分叉与共识](chapter-mining/forks-and-consensus.md)
    * [孤块，钢与铁](chapter-mining/orphan-block.md)
    * [去中心化的迷思](chapter-mining/myths-of-decentralization.md)

-----
* [比特币安全](chapter-security/README.md)
    * [安全指南](chapter-security/security-policy.md)
    * [比特币热钱包](chapter-security/hot-storage.md)
    * [比特币冷钱包](chapter-security/cold-storage.md)
    * [比特币硬件钱包](chapter-security/hardwallet-storage.md)
    * [安全拆分密钥技术](chapter-security/secure-split-key.md)

-----
* [比特币客户端](chapter-client/README.md)
    * [稳定的协议](chapter-client/stable-protocol.md)
        * [Open BSV license](chapter-client/open-bsv-license.md)
    * [Bitcoin SV 客户端](chapter-client/bitcoin-sv-node.md)
    * [Nodes On Top](chapter-client/nodes-on-top.md)
        * [SPV](chapter-client/spv.md)
        * [Tokenized](chapter-client/tokenized.md)
        * [Planaria](chapter-client/planaria.md)
        * [BitQuery](chapter-client/bitquery.md)

-----
* [元网 Metanet](chapter-metanet/README.md)
    * [Metanet 的意义](chapter-metanet/why-metanet.md)
    * [Metanet 技术实现](chapter-metanet/specification.md)
    * [Metanet 开发者指南](chapter-metanet/developer-guide.md)
    * [Metanet 的应用](chapter-metanet/apps.md)
    * [Metanet 展望](chapter-metanet/prospect.md)

-----
* [比特币与法律](chapter-laws/README.md)
    * [法律中的比特币](chapter-laws/bitcoin-is-within-laws.md)
    * [乌合之众的共识](chapter-laws/consensus.md)
    * [无政府主义与乌托邦](chapter-laws/anarchism-and-utopia.md)
    * [匿名与隐私](chapter-laws/anonymity-and-privacy.md)
    * [数据载体](chapter-laws/data-carrier.md)
    * [无需许可的迷思](chapter-laws/myth-of-permissionless.md)
    * [纯粹的资本主义](chapter-laws/pure-capitalism.md)
    * [Token 与证券](chapter-laws/token-and-security.md)
    * [电子合同与签名](chapter-laws/signature-in-electronic-contract.md)
    * [清算与结算](chapter-laws/clearance-and-settlement.md)
    * [私钥与身份](chapter-laws/keys-and-identities.md)

-----
* [比特币的经济学](chapter-economics/README.md)
    * [钱是一种度量尺度](chapter-economics/money-is-measuring-stick.md)
    * [比特币的供给](chapter-economics/money-supply.md)
    * [减半机制](chapter-economics/halving.md)
    * [手续费与流通](chapter-economics/fees.md)
    * [通缩货币？](chapter-economics/deflation.md)
    * [万链归一](chapter-economics/the-global-chain.md)
    * [小额支付](chapter-economics/micropayment.md)

-----
<!-- * [致谢](acknowledgements.md) -->
<!-- * [词汇表](GLOSSARY.md) -->
<!-- * [更新日志](changelog.md) -->

-----
* [附录](appendix/README.md)
    * [扩容纷争](appendix/history-of-onchain-scaling.md)
    * [中本聪时间线](appendix/time-line-of-satoshi.md)
    * [BitcoinSV 常见问题](appendix/faq-of-bsv.md)
    * [比特币白皮书](appendix/bitcoin-whitepaper.md)
