# Rebirth of Bitcoin: Back to Genesis

![「比特币重生：重返创世纪」计划](images/bitcoin-forks.png)

比特币诞生至今，已经过了十个年头。在此期间，因比特币几年的[扩容争议](http://1bsv.cn/%e6%af%94%e7%89%b9%e5%b8%81%e6%89%a9%e5%ae%b9%e7%ba%b7%e4%ba%89/)无果，比特币分叉为  Bitcoin Coin 和 Bitcoin Cash，随后又分叉为 Bitcoin ABC 和 Bitcoin SV。

尽管比特币的 Ticker 多次变更，Bitcoin SV，即原版的比特币，旨在恢复原始比特币协议，保持其协议稳定且允许其大规模扩容。Bitcoin SV 将秉承维护中本聪在2008年白皮书《[比特币：一个点对点电子现金系统](https://bitcoinsv.io/bitcoin/)》中所阐述的愿景。

> “比特币的本质是，第0.1版一经发布，其核心设计即已固定，并在整个生命周期中保持不变。”
- <cite>中本聪（Satoshi Nakamoto)</cite>

尽管中本聪很早就确定了比特币经济激励和自由竞争的核心设计的本质，太多的人认为比特币协议不够完美而尝试在上面画蛇添足。尽管事实上，中本聪的设计一直都未能得以完全实现，比特币还有着无限的潜力等待着人们发掘。

本项目为「[比特币重生：重返创世纪](https://github.com/imcoddy/rebirth-of-bitcoin)」计划，旨在阐述比特币原始设计的来龙去脉，展现协议稳定与去除人为限制的意义，让读者能以此书为入门参考，了解比特币(BSV)的应用与开发，领会中本聪设计的哲学思想，发掘其中鲜为人知的精妙之处，更好地迎接 2020 年 2 月的「[创世纪](https://bitcoinsv.io/2019/04/17/the-roadmap-to-genesis-part-1/)」升级计划的到来。

前天，在顺利实现 Quasar 升级之后，Bitcoin SV 在主网上打出了 [256M 的区块](https://blockchair.com/bitcoin-sv/block/593164)，正如当年中本聪所说的那样：

> “现有的Visa信用卡网络每天在全球范围内处理约1500万笔互联网交易。比特币仅用现有的硬件就已经可以达到更大规模，而且成本只是信用卡交易的一小部分。比特币从未真正达到规模上限。”
- <cite>中本聪（Satoshi Nakamoto)</cite>

Bitcoin SV 正在成为比特币应有的样子，愿这条路上你也能一起前行。

邱少贤[@imcoddy](https://github.com/imcoddy)

记于 20190730

## 编写指南

### 本地预览

本书可用 Gitbook 编译生成网页版浏览阅读，命令如下：

```
$ npm install -g gitbook-cli
$ gitbook install
$ gitbook serve
```

随后可打开 [http://localhost:4000/](http://localhost:4000/) 浏览。

### 文档编辑

本项目使用 Markdown (更准确地说是 [GitHub Flavored Markdown](https://github.github.com/gfm/), GFM) 的格式编写，可选择 [Typora](https://typora.io/) 或者其它的 Markdown 编辑器进行修改。对于 Markdown 不了解的请参阅这份[简明指南](https://www.markdown.cn/)。

本项目还支持使用 mathjax 公式渲染。由于 GitHub 本身不对公式进行渲染，无法正常显示的可以安装 [GitHub with MathJax](https://chrome.google.com/webstore/detail/github-with-mathjax/ioemnmodlmafdkllaclgeombjnmnbima) 插件，以获得良好的阅读体验。

### 排版指南

本项目建议参照[中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines)的排版规范，主要采种下列几点：

* 中英文之间需要增加空格
* 中文与数字之间需要增加空格
* 全角标点与其他字符之间不加空格
* 数字使用半角字符
* 专有名词使用正确的大小写

### 图片添加

图片统一添加到项目根目录的 `images` 文件夹，命名方式根据图片的内容，以全小写英文的方式命名，如 `double-spending-problem.png`。如果需进一步区分，可增加所在章节的前缀，如 `transaction-double-spending-problem.png`。

引用图片的方式建议增加 Alt Text，如 `![Double Spending Problem](/images/double-spending-problem.png)`

由于 Markdown 默认显示全宽图片无法指定大小，如果需要特别指定图片大小的场景，建议使用 HTML 代码来指定大小和居中布局。 `<img src="/images/double-spending-problem.png" width = "400" alt="图片名称" align=center />`

## 提交流程

本项目欢迎读者直接在 GitHub 上提交勘误，也可以发至邮箱 imcoddy@gmail.com 反馈。

请先将本项目 fork 至自己的 GitHub 帐户里面，并添加本项目为 `upstream` 以便能及时获取最近的更新：


```
$ git remote add upstream https://github.com/imcoddy/rebirth-of-bitcoin.git
```

每次编辑时从 `develop` 新建 `feature/summary-of-modification` 的新分支提交 Pullrequest。具体的操作方式如下：


```
$ git checkout develop
$ git pull upstream develop
$ git checkout -b feature/new-summary-of-modification (自行根据需要修改为合适的名称)
```

提交时直接将 `feature/summary-of-modification` 向 `imcoddy/rebirth-of-bitcoin` 的 `develop` 分支提交 Pullrequest 即可。

## 版权说明

[![Creative Commons License](https://mirrors.creativecommons.org/presskit/buttons/80x15/png/by-nc-nd.png)](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)

Rebirth of Bitcoin: Back to Genesis 《比特币重生：重返创世纪》一书以 [署名-非商业性使用-禁止演绎 4.0 国际 (CC BY-NC-ND 4.0)](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)  的许可协议发布，并预计于 2021 年前(即 Bitcoin SV 的 「[创世纪](https://bitcoinsv.io/2019/04/17/the-roadmap-to-genesis-part-1/)」升级后的一年之内)以 [署名-相同方式共享 4.0 国际 (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/deed.zh)  方式开放许可授权。
