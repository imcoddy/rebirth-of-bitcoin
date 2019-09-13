# Planaria
# Bitcoin SV 的开发哲学 —— 变形虫框架

[Source](https://zhuanlan.zhihu.com/p/62287840 "Permalink to Bitcoin SV 的开发哲学 —— 变形虫框架")

之前的心路历程的第二篇由于需要收集和准备的材料很多，加上工作限制时间有限，之后再发布。

## 前言

与币圈其他的浮躁气氛不同，现在 BSV 的社区是一个非常有创造力，非常有活力的社区。在分叉后不到半年的时间里，各种应用如雨后春笋一般涌现出来。我早期研究比特币的时候，设想了很多比特币支付功能以外的扩展应用，可惜受限于区块容量，脚本制约等等因素无法实现，而 BSV 在回归中本聪版本的比特币的路上，开放了很多限制，让曾经无法实现的想法得以实现，也极大的促进了生产力的提升。有点类似中国的改革开放，放开限制，让资本、技术、开发者自由加入，充分竞争，成就了中国经济的奇迹。

目前 BSV 的应用大多集中在信息的上链存储，得益于巨大的区块容量（以后还将坚定不移的扩大，直到 Unlimited），链上的储存成本之低已经碾压所有主流公链，之后随着区块进一步扩大，这个成本还会降到更低。如果上传一篇文章到链上，只需几分钱的矿工费，就可以让你的文字永存于世上。按照 1sat/byte 的费率计价（如果矿工接受，这个费率可以更低），上传 1M 的内容上链大概需要 0.01 个 BSV，按目前的价格 500 人民币的币价来算，大概需要 5 块钱。乍一看 5 块钱 1M 的成本跟 AWS S3 这样的中心化云服务储存桶相比简直是高的可笑，但是加上以下我说的这几个条件，这个成本就会变得非常合理而且难以置信的低。

1. 数据需要绝对安全的保存，不允许篡改或毁坏，安全级别达到 11 个九（99.999999999％）
2. 数据要保存 100 年以上
3. 数据可以随用随取，读取数据成本很低（参考 AWS Glacier 查询归档数据的成本）
4. 一次付费，永久保存（无需续费）
5. 不需要注册和身份认证，免许可，任何人都有上传的自由

综合以上的分析可以知道，区块链技术在信息安全存储的方面具有所有传统中心化云存储所不具备的安全优势和成本优势。正是这种成本优势，才能引入大规模的商业应用，才能在此基础上构建更多功能复杂的应用。

## Unwriter

BSV 社区的开发者有很多大牛，其中我最佩服的就是 Unwriter，不止因为技术层面，更是因为立场和思想层面。unwriter 是一个匿名的开发者，从未接受 BCH 或 BSV 社区的捐赠，是一个保持经济中立的独立开发者。他的目的很纯粹，就是利用区块链的技术，来实现伟大的产品。而在 BCH 和 BSV 分家之后，他写了一篇文章叫做《The resolution of the Bitcoin Cash experiment》，表明了立场，开发者才是一条公链项目最大的财富，开发者才是资本家。

> The application developers are the “capitalists” of the “nation” of Bitcoin. And THEY are the ones who determine the “wealth of the nation”.

公链的价值来源于应用，而应用不是凭空产生的，是开发者开发出来的，最能吸引开发者的公链，最能吸引应用。作为开发者，最不希望的就是把自己的应用建立在一个根基不稳定的系统之上，如同 BCH 这种半年一大改的链，没有人希望之前的开发成果随着一次可有可无的升级而全部重构。

这篇文章对我的影响很大，我本身也是开发者，对分叉当时 ABC 的所作所为看在眼里，记在心上，这也是我全面支持 BSV 的很大的一个影响因素。

英文原版如下：

[The resolution of the Bitcoin Cash experiment​medium.com](https://link.zhihu.com/?target=https%253A//medium.com/%2540_unwriter/the-resolution-of-the-bitcoin-cash-experiment-52b86d8cd187)

还可以参考中文版《深度解析比特币现金实验》（这里感谢黄酥酥的翻译）：

[Sina Visitor System​www.weibo.com](https://link.zhihu.com/?target=https%253A//www.weibo.com/ttarticle/p/show%253Fid%253D2309404312035490985667%2523_0)

Unwriter 的最重要的作品就是 BitDB，这是一个曾经开发在分叉前的 BCH 上的持久层框架，它能够将比特币区块链映射成一个 MongoDB 的实例，支持通过 MongoDB 的查询语句来查询区块链。一句话概括，就是这个框架将区块链当做数据库来使用，在分叉之后，Unwriter 放弃在 BCH 开发，将所有的项目转向 BSV。

在 BitDB 的基础之上，衍生出来了很多变种的数据库应用，挑几个举举栗子（日后有时间会逐个总结一下这些数据库应用的使用方法和应用场景）

## BitDB 变种

Genesis（创世纪）：一个用来支持标准地址查询全量交易的数据库，可以看做一个全量区块链浏览器

[A Bitcoin SV Exclusive BitDB node​medium.com](https://link.zhihu.com/?target=https%253A//medium.com/%2540_unwriter/genesis-a25b121e0575)

Babel（巴别塔）：一个专门用来存储和查询链上应用数据的数据库，可以看做 Genesis 的一个子集，只关心数据体不关心交易内容。

[A BitDB Node for Data-Only Bitcoin Applications​medium.com](https://link.zhihu.com/?target=https%253A//medium.com/%2540_unwriter/babel-230f73ed5dcb)

Chronos（克罗诺斯 —— 时间之神）：一个专门使用时间戳来操作和查询数据的数据库，查询规范与前两个不太一样，专门用来处理时间相关的应用。

[An Ephemeral BitDB for Dealing with Time​medium.com](https://link.zhihu.com/?target=https%253A//medium.com/%2540_unwriter/chronos-f0f751669fef)

Meta（元）：一个对区块元数据进行操作和处理的数据库，只关心区块，不关心区块内部的交易。

[A BitDB for Bitcoin Block Metadata​medium.com](https://link.zhihu.com/?target=https%253A//medium.com/%2540_unwriter/meta-be3c18582ec7)

Bitsocket（比特套接字）：一个监听区块链事件，由事件触发业务逻辑的推送消息总线框架。

[A Programmable Bitcoin Push Notifications API​medium.com](https://link.zhihu.com/?target=https%253A//medium.com/%2540_unwriter/bitsocket-e7feb4f837ad)

Planaria（变形虫）：一个支持自由定制规则的储存框架，在此基础上可以实现所有的程序逻辑，这也是本文要重点说明的理论基础。

[Opening up a whole new dimension to programming Bitcoin powered state machines​medium.com](https://link.zhihu.com/?target=https%253A//medium.com/%2540_unwriter/Planaria-systems-programming-interface-e9db226012d5)

## 变形虫 Planaria

上文中举例的各种特殊的数据库应用都可以归为具有某种特定实现的变形虫。变形虫这个框架阐述了如何基于比特币的存储功能，来实现几乎所有的应用程序的哲学逻辑，也是论证 Bitcoin Maximalist 一直在信仰的比特币可以实现世间万物的逻辑基础。

有些人说 ABC 和 BSV 没有什么本质区别，都是大区块币，这些人恰恰只能看到表象，看不到本质。ABC 和 BSV 的核心分歧在于中本聪比特币本身的设计是不是完备的，ABC 认为比特币的设计是老旧的，功能残缺的，需要各种炫技的新技术（无论有没有用），需要半年一次的迭代更新，一次一次进行底层的修改来追赶热点。BSV 则认为，比特币的设计已经很完备，底层逻辑无需改变，需要放开各种无谓的限制，让应用在基盘之上自由生长。之前关于 CTOR 的问题就有人论证过，使用比特币已有的脚本操作码就可以实现这个功能，而 ABC 却不惜以分叉为代价来强行加入这个本就可以实现的功能作为底层的一部分，彻底破坏了矿工按照交易体积来计价收费的逻辑，强行把比特币变成以太坊（按照不同的操作码进行 gas 计价，再乘上 gas 单价来算出手续费）。

我认为之所以 Unwriter 在变形虫的框架文档中介绍了很多看似是哲学以及毫不相干的内容（又是状态机，又是薛定谔的猫），实际上就是在证明比特币可以实现图灵完备，可以实现复杂的智能合约。

最近我希望通过 BSV 的基础设施来实现一些小的功能应用，所以一直在研究现有的框架，变形虫就是研究的一个重点，说实话，现在我也不敢完全宣称了解了其中的全部思想，但就以我粗浅的理解来看，Unwriter 所描述的，正是 CSW 所说的 MetaNet 万链归一的可能性，从理论到实际使用，都没有背离最初支持 BSV 所希望的愿景。

文章如下：我尽量以自己的理解来简单阐述一下

[Planaria Introduction​docs.planaria.network](https://link.zhihu.com/?target=https%253A//docs.planaria.network/%2523/intro)

## Bitcoin SV 的开发哲学

无极生太极，太极生两仪，两仪生四象，四象成八卦，八卦演万物。

看似是玄学的话，却是计算机最基本的事实。我们现在所有复杂的计算机程序，应用，图片，视频等等等等归根结底就是 0 和 1 的组合，换句话说，所有的数据都可以被划归成 0 和 1，也就是文件。

> Everything is a file.
> If you think about it, everything runs on a file. And all applications are just files, which make use of other files.
>
> Once you see it this way, you’ll realize that Planaria can be used to build all kinds of full fledged Bitcoin-powered computation backends.
>
> Below I’ve listed some interesting technologies that are all built on files just to help with imagination.
>
> But think about it. EVERYTHING IS A FILE.

这里的文件不是简单意义上的储存在硬盘上带后缀的 “文件”，而是一切数据的统称。存在内存里的，缓存里的等等等等。对数据进行的处理称为操作，操作依赖代码，代码本身也是文件。

BSV 区块链可以低成本存储文件，而利用变形虫框架可以对文件进行增查删改。很多人质疑这一点，文件一旦上链不就永不可更改了吗？这点没有错，但是如果跳出这个上链的文件本身来看，上链的文件只是这个文件在一个固定的时间戳的快照，快照是永不更改的，而文件是可以更改的。类比一下，我们活着其实也是被时间定格在一个个的快照中，过去的自己没有办法更改，但是现在的自己可以改变，我们随时随地可以产生新的快照，已经产生的快照作为我们的历史被定格，未来却是可以改变的。上一秒的我和这一秒的我还是同一个我吗？

这种文件通过快照来更新自己的模式我们并不陌生，Git 就是一个最直观的例子。通过 Git 系统，也可以对文件进行增查删改，而且 Git 存储文件与传统文件存储不同的地方是，Git 记录了一个文件的所有变更历史，可以追踪溯源回滚等，传统文件存储用新状态替换掉旧状态，没办法保留历史。Git 通过一次次的提交（Git commit）来修改文件的最新版本，最新版本就是最新的状态，而一次次的提交被 Branch（分支）定格在历史中，这种模式和比特币的文件管理简直是天作之合，比特币的文件管理从安全性和身份管理上甚至更为优越，还自带了价值传递的功能，不夸张的说，比特币 BSV 的文件系统可以看做一个超级 Git。

Unwriter 在文中，将比特币称为发动机（motor），将比特币看做动力源，源源不断的由矿工提供交易和事件（驱动力）。并对这种动力源进行了一些要求：

> The rotation logic (Bitcoin's algorithm) is deterministic and secure (powered by Proof of Work), making it the perfectly stable piece of technology to power all kinds of useful machines.

简单解释一下，这种动力源要可预测（比如汽油的燃烧产生的动力，是可控的），动力源要安全（由 PoW 保证），动力源要稳定。这样就可以驱动机器去运转。

Unwriter 将应用程序称为机器（machine），机器需要动力源来驱动，而相同的动力源可以根据机器的构造不同，产生不同的效果。比如同样品质的汽油，在不同的车上的能耗比不一样。有了一个稳定的动力源，就可以以它为基础构造各种各样的机器，来实现各种各样的功能。

### 有限状态机（[Finite State Machine](https://link.zhihu.com/?target=https%253A//docs.planaria.network/%2523/intro%253Fid%253Dfinite-state-machine)）

文中阐述了使用 BSV 构建有限状态机。什么是有限状态机？这是一个计算机的术语，用于描述一种具有有限个状态，并能在这些状态中进行切换的机器（或程序）

> ** 有限状态机 **（英语：finite-state machine，[缩写](https://link.zhihu.com/?target=https%253A//zh.wikipedia.org/wiki/%25E7%25B8%25AE%25E5%25AF%25AB)：**FSM**）又称 ** 有限状态自动机 **，简称 ** 状态机 **，是表示有限个 [状态](https://link.zhihu.com/?target=https%253A//zh.wikipedia.org/wiki/%25E7%258A%25B6%25E6%2580%2581) 以及在这些状态之间的转移和动作等行为的 [数学模型](https://link.zhihu.com/?target=https%253A//zh.wikipedia.org/wiki/%25E6%2595%25B0%25E5%25AD%25A6%25E6%25A8%25A1%25E5%259E%258B)。

比如 unwriter 在文中说明的地铁闸机，闸机有两种状态，一种是锁定（人不能通过），一种是解锁（可以通过），而输入的动力源种类也有两种，一种是刷卡，一种是推动。闸机可以根据输入的动力源来切换自己所处的状态。如果闸机锁定，此时刷卡，闸机就会变成解锁状态，而如果此时推动，则继续锁定。

将一个有限状态的数据作为机器，给其一定的动力源作为输入（input），就会产生一定的输出（output），输出取决于输入和机器的处理逻辑，这就是计算机函数（function）或者方法（method）的基本形态。

将 BSV 的链上交易作为动力源，将已经上链存储的数据作为机器，就可以推动任意的程序来运转，这就是 BSV 智能合约的理论基础。

### 无限状态机（[Infinite State Machine](https://link.zhihu.com/?target=https%253A//docs.planaria.network/%2523/intro%253Fid%253Dinfinite-state-machine)）

上文提到的有限状态机已经可以帮助我们事先几乎所有的应用程序，但是比特币可以实现的远不止这些。下文脑洞略大，但并非胡说八道，有研究和理论的基础。

Unwriter 用薛定谔的猫（同时是生和死两种状态的叠加）来形容这种状态机，这也是这篇文章让人最难于理解的地方，这种同时具有多个甚至可以相互矛盾的状态的现象不止是理论，而且真实存在。量子物理的世界就充斥着这种矛盾和和谐。

薛定谔的猫是什么问题，大家想必都有所了解。这里我们还是针对状态机来说明。

**we can build applications that are NOT constrained by resources.**

** 我们可以构造出一种不被文件资源所限制的应用 **

We can build machines that:

我们可以构造出满足以下条件的状态机：

* are running and not running at the same time（在同一个时间处于运行与关闭的叠加态）.
* don't have to exist nor run today, yet can be regarded as "running" today（并不需要现在就在运行，却可以被看做一直在运行）.
* can have multiple parallel states that even contradict with one another（可以同时具备多个平行的状态，甚至状态之间互相矛盾）.
* never die, even when they are not alive（从来不会死，即使从来没有被启动过）.
* can interact with one another in non-deterministic ways（可以和其他的状态机之间以一种 “不可预测” 的方式交互）.
* can interact with one another in combinations which even its original inventors never foresaw（可以和其他状态机相互组合，产生连原作者也无法预料到的神奇现象）.
* spawn other such machines without external intervention（可以在没有外界影响的情况下，孵化出其他的状态机）

Unwriter 说实现这个需要彻底放开所有的脚本限制。在我看来，这种类型的程序，更像是无法预测状态的程序，量子力学有提到，这个世界是测不准的，存在大量的真随机。而所有生物的产生和演化更符合上述提到的测不准现象，这就有可能在比特币系统上衍生出原始计算机生物，进而随机地衍生出高等计算机生物（人工智能），之前记得在那里看到过，人脑也是没办法用计算机模拟的，因为人脑是测不准的。而无限状态的程序提供了这样一种衍生出全宇宙可能（抱歉这里脑洞太大，已经逼近我的理解和认知的边界了）。

而 Unwriter 提到的同时是死和活两种状态的程序，可能是有一部分 serverless service（无服务器服务）的意思，一个程序只要部署上链，不需要你运行它（此时是死的），但它的状态又时时在改变（又是活的），直到你下次运行它，才能确定它最终的状态。就像只有打开盒子看到猫才能确定薛定谔猫的状态，而打开盒子就破坏了这种同时死同时活的状态。

### Planaria 概括

1. 一个比特币应用程序的框架，具有以下特点

* 透明性：数据都是从区块链上获取到的，绝对安全可信
* 便携性：变形虫框架利用了 docker 这种优秀的容器技术，让部署变得更加简单
* 共享性：区块链本身相当于一个超级云平台，平台上的应用之间可以相互共享数据
* 可定制化：对变形虫的定制没有限制，你可以实现任何想实现的程序
* 用户友好：变形虫框架将 raw 交易进行封装，将用户可读的数据展现出来

2\. 独立可定制的 CRUD（增查删改）数据库

* 使用 MongoDB 作为可读实例
* 提供 CRUD 的 api
* 读写分离，MongoDB 只读，而写操作需要构造比特币交易上链
* 由区块链事件驱动数据更新
* 自带权限认证功能（比特币交易本身即是权限认证），鉴权秘钥无需服务器保管

3. 由比特币作为动力源驱动的有限状态机

* 输入： onmempool，onblock，onrestart 三个 api 方法，来监听不同的区块链事件，来进行相应的处理
* 处理逻辑：数据的 CRUD
* 状态存储：Planaria and Planarium 两个组件，一个爬行区块链，一个提供查询
* 输出：通过 zeroMq 消息队列，将结果推送到消息订阅者

4\. 无限状态机（理想状态）

实现真随机和测不准的程序。实现无需长期运行的 serverLess 程序。实现人工智能。脑洞略大，此处需要继续研究实现方案）

### 变形虫的开发范式

基于区块链开发的应用，和传统的 Server Client 类型应用在一些认知上有些差异，理解这些差异正是理解区块链颠覆式创新的核心。

1. 分布式鉴权系统

传统的身份认证，需要用户在服务器上存储账号密码（密码 hash），并由中心服务器进行权限校验，再授权相应的操作。

而区块链应用，操作即交易，交易即鉴权。能发起相应的交易，自然有权限。直接的结果就是，服务器不再保存秘钥，而将秘钥交还给用户手中，由用户自己保管，破解了中心化系统长久存在的秘密泄露威胁。

2. 读写解绑

传统服务器，读写全部都是由中心服务器进行，如果服务器被攻破或内部人员作恶，则可能产生一些违反用户意愿的数据篡改，数据删除等行为。变形虫框架将写权限完全归还用户，用户通过构建特殊格式的交易来写数据库，而中心服务器只负责监听区块链，提供给用户数据。这样，就算中心服务器宕机，用户写入数据，修改数据的功能还能完整保留下来（能发交易就能改），这可能就是 Unwriter 说的又死又活的状态吧。

3\. 独立应用

传统的应用，如果应用的开发者或者维护者停止了应用维护，则用户就算想要接着使用，都没有办法改变现状。典型的就是 360 云盘这样的，强制给用户一个截止日期，然后就清空所有云盘中的数据，停止对外提供服务。

而通过 Planaria 构造的程序，代码和用户数据都被部署到了链上，用户不需要再关心开发者或者维护者的态度，这个程序会永远存活下去。任何人都可以随时随地执行这些程序，让这个应用永远活下去。用户的数据也永远保留在链上，不会丢失。

4\. 比特币作为消息总线

传统的应用，生态都是封闭的，最直观的例子就是微信和支付宝各自的生态闭环。在其生态内，可以共享生态内的资源，而跨生态之间的合作就会变得很麻烦。

而比特币应用共享同一个区块链系统，在其上可以开发出各种应用之间的业务合作，让消息的共享，价值的流动更加通畅，可能会爆发出更多更好玩的应用。

同时，由于比特币的开放性和免许可性，应用之间的通信可以免受各种和谐政策的影响，点对点发送，自由无限制。

5\. 透明计算

和以太坊的智能合约相同，任何人都可以下载，执行，校验程序的执行过程和执行结果。不一样的是，以太坊的所有节点无论愿意与否，都要全部执行一遍智能合约，导致以太坊的可拓展性非常差，又慢又贵。而变形虫程序，只有关心合约的人需要下载合约来执行，其他人可以无视与自己无关的程序。加上比特币的 UTXO 结构，在拓展性上远胜于以太坊的账户模式，这个问题以后专门写文章讨论。

6\. 代码永恒

长期来看，所有中心化的程序都会死亡，所有生物都会死亡，只有基因永存。而这个基因，就是被记录到区块链上的代码，代码永恒。当你下载代码执行的时候，相当于你承载了这个基因存活了下来，人类归根结底也是基因的载体。

## 结语

这篇文章主要从理论方面介绍了 BSV 区块链上的可能性，BSV 的理想不是什么 IXO 这么肤浅和浮躁的目标，BSV 的目标是改变商业模式，改变互联网模式，改变人类模式。很多人现在在看 Metanet 的笑话，但是 who cares，haters gonna hate。在你们沉浸在无休止的谩骂和诋毁的时候，BSV 正在建设基础设施，而这些基础设施就是以后让你们的山寨币归零的最强大的机器。

昨天出了个事，这里有必要提一下，币安的赵长鹏宣称要下架 BSV。且不说作为交易所无法保持中立的立场对不对，就这个幼稚的想法就很可笑。市场是开放的，你 Binance 下架 BSV，最高兴的是谁？不是 Core 党，也不是 BCH 党，而是火币和 OKex，你做交易所的不是在跟社区竞争，而是在跟其他交易所竞争，拜托你搞清楚对象。况且 BSV 社区压根不 care 你 binance 上架还是下架，当初上架不也没人给你们上币费吗？BSV 现在正在做的就是建设，就是真正脚踏实地地做应用，真正到了人人都在用 BSV 的时候，根本就不愁交易所上不上架的问题好吗？你下架，伤害的只有你自己，你不想挣的手续费有的是人想挣。做事情没必要做的太绝，免得以后没得台阶下。

得益于现在的文章上链应用已经非常的成熟和好用，我的所有文章都会先上传到 BSV 的区块链上，花上个几分钱就能永久留存著作权，真的是非常棒。欢迎转发本文，但是用作商业应用，请和我联系，我已经保留了最硬核的著作权证据，也将身体力行去享受着 BSV 应用带来的便利性。欢迎大家来我链上的文章打赏。

[On-chain text and file sharing​www.bitpaste.app](https://link.zhihu.com/?target=https%253A//www.bitpaste.app/tx/b4f0b5ff7de49a13cc8db5b72c07f6179294eeddec117e147420ad21f5a692fe)


变形虫 Planaria 是 Unwriter 大神基于 bitdb 在 BSV 链上开发的一个可编程化的持久层框架，关于变形虫的特点和编程思想，可以参考我之前的文章，强烈建议先看完前文再阅读本文。

[Bitcoin SV 的开发哲学 —— 变形虫框架](https://zhuanlan.zhihu.com/p/62287840)

这篇文章从纯技术的角度总结一下使用变形虫框架进行开发所需要的注意的一些点和踩过的一些坑。

Unwriter 大神在变形虫的技术支持文档中已经很详细的写了如何去部署和搭建一个节点，以及如何去调用节点提供的接口来实现应用程序的功能。我自己也照着文档去尝试着搭建节点，在搭建的过程中也遇到了一些坑，也遇到过各种各样的异常情形不能成功运行节点，也为了解决这些异常去详细地研读过源代码。现在将一些心得和学习记录总结出来分享。

一般使用 Planaria 实现一个 BSV 的链上应用需要以下的一些步骤：
1. 搭建一个变形虫的实现节点（machine），根据业务逻辑，编写变形虫的 Planaria（面向区块链的爬虫）和 Planarium（面向人类的接口），决定变形虫如何去爬行区块链，如何规范化地将链上数据存储为可读数据。这是一个变形虫开发的重点，之后专门学习总结如何编写 planaria.js 和 planarium.js。
2. 客户端使用比特币的 SDK 构造特定格式交易来生成应用数据，广播这些交易让数据上链。然后由变形虫去监听这些数据并储存到 MongoDB 中提供给客户端使用。构造交易也是应用开发的重点，可以借助各种已有的库，比如 bitcoinj-sv，比如 money button 维护的 BSV.js，或者使用 unwriter 大神封装好的 datapay.js
3. 客户端使用刚才搭建好的节点，或者使用第三方已经提供的节点，使用 bitquery 查询语句调用节点的 api，从节点的 MongoDB 中获取链上数据或者监听区块链事件
4. 客户端获取到数据或者监听到事件之后，采取相应的动作，此处的开发就和一般的应用没有区别了

## 节点搭建和部署
如果想定制化的开发变形虫，指定变形虫的特征和行为，就需要自行搭建节点，自行编写 Planaria 和 Planarium。

因为搭建和维护一个节点目前需要相对较高的成本，如果应用本身不需要定制化的变形虫，可以使用 unwriter 已经公开的一些 endpoint。这些已经实现了和部署了的节点包括 genesis，chronos，babel，c 协议，以及新出的 file-server 这些变形虫的变体节点（这些变体节点的功能在之前的文章有说明），可以根据自身的需要去调用这些公开 api。如果不清楚具体需要哪一种功能，可以调用 genesis，具备全量的且相对原始的数据。

[已经公开注册的变形虫节点](https://planaria.network/)

目前版本的变形虫需要将 BSV 全节点和变形虫节点置于同一台服务器（这是代码在配置上的原因，Planaria 在代码中共用了 MongoDB 的 HOST 和 bitcoind 的 HOST），应该会在后续的版本上优化。我开始的时候将 BSV 全节点和变形虫节点搭建在两个不同的服务器上，结果吃了不少亏，研究了源码才发现这个问题。

另外为了部署方便，变形虫采用的是 docker compose 进行自动化的容器化部署，首次部署时我采用的操作系统是 Ubuntu 18.04，但是在启动 Planaria 容器后一直报异常，深入到 docker 内部查看 npm 的异常日志发现了如下的错误信息，大概是在 build zeromq 的时候出现了问题，照理说 docker 容器应该是可以跨平台使用的，我目前还没有很好的解决掉这个问题，不知道底层的原因是什么。后面更换了操作系统 Ubuntu 16.04，一切正常，我把错误日志贴出来，希望知道原因的朋友多指教。


```
13592 silly saveTree `-- zeromq@4.6.0
13592 silly saveTree   `-- prebuild-install@2.5.3
13592 silly saveTree     +-- expand-template@1.1.1
13592 silly saveTree     +-- minimist@1.2.0
13592 silly saveTree     `-- pump@2.0.1
13593 warn planaria@0.0.1 No description
13594 warn planaria@0.0.1 No repository field.
13595 warn optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.8 (node_modules/fsevents):
13596 warn notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.8: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})
13597 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Valid OS:    darwin
13597 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Valid Arch:  any
13597 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Actual OS:   linux
13597 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Actual Arch: x64
13598 verbose stack Error: zeromq@4.6.0 install: `node scripts/prebuild-install.js || (node scripts/preinstall.js && node-gyp rebuild)`
13598 verbose stack Exit status 1
13598 verbose stack     at EventEmitter.<anonymous> (/usr/lib/node_modules/npm/node_modules/npm-lifecycle/index.js:301:16)
13598 verbose stack     at EventEmitter.emit (events.js:182:13)
13598 verbose stack     at ChildProcess.<anonymous> (/usr/lib/node_modules/npm/node_modules/npm-lifecycle/lib/spawn.js:55:14)
13598 verbose stack     at ChildProcess.emit (events.js:182:13)
13598 verbose stack     at maybeClose (internal/child_process.js:962:16)
13598 verbose stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:251:5)
13599 verbose pkgid zeromq@4.6.0
13600 verbose cwd /planaria
13601 verbose Linux 4.15.0-1035-aws
13602 verbose argv "/usr/bin/node" "/usr/bin/npm" "install"
13603 verbose node v10.10.0
13604 verbose npm  v6.4.1
13605 error code ELIFECYCLE
13606 error errno 1
13607 error zeromq@4.6.0 install: `node scripts/prebuild-install.js || (node scripts/preinstall.js && node-gyp rebuild)`
13607 error Exit status 1
13608 error Failed at the zeromq@4.6.0 install script.
13608 error This is probably not a problem with npm. There is likely additional logging output above.
13609 verbose exit [ 1, true
```

上面两个问题给我开始的部署带来了很大的麻烦，希望后续版本可以改进这些问题。下面说一下节点所需要的配置：

* 至少 2G 的内存
* 至少 300G 的硬盘储存空间（unwriter 写 200G，肯定不够用）

这些配置是为了支撑一个 BSV 的全节点，加上 docker，以及运行在 docker 上的 Planaria 和 Planarium，其中还要运行一个 MongoDB 的服务器。如果机器性能太差，则在同步区块和读写数据库的时候会消耗大量的时间，更可能在 BSV 突如其来的大区块冲击下被搞垮，这也就是为什么 BSV 不提倡使用家庭电脑运行全节点，搭建节点就是为了商用。只是学习和了解的话建议直接使用公开节点 api。

而我的节点搭建在 AWS 上，考虑成本和性能，加上为了研究和学习使用，采用了如下的配置：

* T2 Medium 的 EC2 实例类型（2 vCPUs，4GB 内存）
* 1000GB 的 EBS 磁盘（Megnetic Standartd）
* Ubuntu 16.04 的操作系统

更低的配置我测试过，效果很不好，同步区块校验交易效率很低，读写性能也很受限制，上面的配置的成本每个月大概 100 美元（如果要推广商用，这个成本还要翻 N 倍），所以再次强调，full node is about business。所以如果要开发区块链应用，那么应用的收益至少要能涵盖这个成本，这就是资本主义竞争的残酷，树莓派节点只会拖慢网络，不会服务用户。

机器准备好了之后，就可以开始搭建节点了。英文好的可以直接参考 Unwriter 的文档

[Run a Node](https://docs.planaria.network/#/guide?id=_0-system-prerequisites)

### 安装 bitcoin sv 全节点客户端
BSV 全节点没有可视化客户端，只能通过命令行来安装和运行，这也一定程度上避免了个人运行全节点来浪费资源。

首先从 bitcoinsv.io 的网站下载客户端软件（版本 0.1.1）：
[官方下载镜像](https://download.bitcoinsv.io/bitcoinsv/0.1.1/)

然后下载名称为 bitcoin-sv-0.1.1-x86_64-linux-gnu.tar.gz 的压缩包到你的程序目录文件下。

解压客户端文件

```
tar -zxvf bitcoin-sv-0.1.1-x86_64-linux-gnu.tar.gz
```

然后在根目录建立 bitcoin 的数据文件夹，准备编辑启动配置 bitcoin.conf。

```
cd ~

mkdir .bitcoin

cd .bitcoin/

vi bitcoin.conf
```

bicoin.conf 是比特币客户端的配置文件，变形虫节点的运行需要依赖 BSV 节点的 index 和 zeroMQ，因此要对其进行相应的配置，我的配置如下：


```
dbcache=4000
# Must set txindex=1 so Bitcoin keeps the full index
txindex=1

# [rpc]
# Accept command line and JSON-RPC commands.
server=1
# Default Username and Password for JSON-RPC connections
# Planaria uses these values by default, but if you can change the settings
# When you run 'pc start'
rpcuser=YOURNAME
rpcpassword=YOURPASSWORD
# If you want to allow remote JSON-RPC access
rpcallowip=0.0.0.0/0
rpcbind=0.0.0.0
# [wallet]
disablewallet=1

# [ZeroMQ]
# ZeroMQ messages power the realtime Planaria crawler
# so it's important to set the endpoint
zmqpubhashtx=tcp://0.0.0.0:28332
zmqpubhashblock=tcp://0.0.0.0:28332

# Planaria makes heavy use of JSON-RPC so it's set to a higher number
# But you can tweak this number as you want
rpcworkqueue=512

# Support large mempool
maxmempool=4000

# Support large pushdata
datacarriersize=100000

# Long mempool chain support
limitancestorsize=100000
limitdescendantsize=100000

addnode=104.215.14.250
addnode=13.231.139.183
addnode=13.231.20.63
addnode=13.231.92.219
addnode=54.95.24.226
addnode=52.195.19.127
```
最后的几个 ip 是我手动添加的距离比较近的 sv 节点 ip，因为 BSV 和 bchabc 在网络层没有做隔离，在首次同步的时候经常会链接到分叉前的 abc 的节点，在同步到分叉高度后就会因为不兼容而无法继续同步，所以我手动添加了一些节点来帮助我同步数据，更快地找到组织。寻找这些节点也很简单，打开 blockchair 的节点接口，就可以查看现有的的节点了，选取 `"version": "/Bitcoin SV:0.1.0 (EB128.0)/",` 这样的节点 ip 添加到 conf 中。

[blockchair nodes](https://api.blockchair.com/bitcoin-sv/nodes)

然后切换到 bitcoin-sv 的 bin 目录下，启动客户端，可以用 deamon 模式也可以 nohup 挂载。

```
cd bitcoin-sv-0.1.1/bin/

sudo nohup ./bitcoind &

./bitcoin-cli getinfo

```
如果 bitcoin-cli 成功返回以下结果，说明全节点启动成功

```
{
  "version": 100010100,
  "protocolversion": 70015,
  "blocks": 581047,
  "timeoffset": 0,
  "connections": 36,
  "proxy": "",
  "difficulty": 85497465595.02908,
  "testnet": false,
  "stn": false,
  "paytxfee": 0.00000000,
  "relayfee": 0.00001000,
  "errors": "",
  "maxblocksize": 128000000,
  "maxminedblocksize": 32000000
}

```

### 安装 docker
变形虫框架依赖多个组件运行，unwriter 为了方便人们部署变形虫，将这些组件整合在容器中，这样就具备了便携性和跨平台的特性。

安装 docker 请直接参考 docker 的官方文档或者在网上搜索安装方法，这里不赘述。

[Ubuntu 下安装 docker 的官方文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

### 安装 docker compose
docker compose 是用来将多个容器进行管理和协调的工具。安装同样不赘述。

[Install Compose on Linux systems](https://docs.docker.com/compose/install/)

### 安装 node
虽然变形虫等框架是运行在 docker 内，docker 内集成了 nodejs 等环境，但是如果使用 Unwriter 提供的 Planaria Computer（后文简称 PC）工具来管理各个组件容器（这样可以不用直接跟 docker 打交道），就需要在机器上安装 PC 所需要的 Node 环境。
Planaria Computer 提供了一系列命令和 cli 工具来轻松管理，发布，下载，部署，维护变形虫节点，因此我们安装 PC 之前需要安装其运行的 NodeJs 环境。

ubuntu 下安装 nodeJs 很简单，可以用 nodesource 的源来安装

```
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```

安装后执行 `node -v` 如果返回 `v12.0.0` 就说明安装成功。

### 安装 Planaria Computer 与启动变形虫

```
npm install -g Planaria
```

安装好之后，就可以开始创建变形虫的状态机（machine）了。

先给 Planaria 创建一个文件夹，作为变形虫节点的数据文件夹：


```
mkdir Planaria

cd Planaria/
```

然后创建第一个状态机

```
pc new genesis
```
这里为了简单起见，我们创建了一个 genesis（创世纪）的状态机，如果要创建自定义状态机，则执行下面的指令（推荐学习者使用 genesis 的状态机，获取全量数据并方便后面的 bitquery 查询）：

```
pc new machine
```

注意，如果创建的是 Genesis 这种已经定制好的状态机，则不需要编写内部的 js 代码，如果创建的空的状态机，则需要编写 js 代码来实现自定义的状态码，这部分技术之后总结。

我们创建好了一个 Genesis 的状态机之后，PC 会生成一个文件夹，名称叫 genes（基因），这里储存了全部的状态机，每一个状态机有一个独特的比特币地址作为文件夹名称，正是因为变形虫采用地址作为状态机的基因（特征标志），状态机理论上就可以有比特币地址那么多的个数，而且同一个地址的状态机可以更新换代，这就涉及到另一个关于 Bitcom 协议的概念了，后文会说。我们进去其中一个状态机的文件夹之后就可以发现如下的几个文件：


```
-rw-rw-r-- 1 1001 1001  100 Apr 26 16:32 .env
-rw-rw-r-- 1 1001 1001  169 Apr 26 16:32 package.json
-rw-rw-r-- 1 1001 1001 4599 Apr 26 16:34 planaria.js
-rw-rw-r-- 1 1001 1001  514 Apr 26 16:32 planarium.js
-rw-rw-r-- 1 1001 1001    0 Apr 26 16:32 README.md
```

.env 中放置了这个状态机地址和对应的私钥，是证明状态机所有权和用来修改状态机的权限秘钥，建议如果自定制了协议，请务必备份好.env 文件，这是协议和状态机所有权的标志。

package.json 是常见的 nodejs 依赖管理，如果你的变形虫代码逻辑需要依赖一些其他的 nodejs 的库，可以在这里面进行编辑。

planaria.js 是面向区块链的爬虫逻辑，里面需要编写如何解析原始交易，并提取其中的信息储存到 MongoDB 中。

planarium.js 是面向应用的展示层逻辑，将 MongoDB 的数据读取出来，加工成应用需要的格式，返回给应用。

README.md 是你的自定义协议或者状态机的一个说明文档（markdown 格式），在公开发布此状态机的时候会上传此文档，作为描述状态机功能和使用方法的文档。

然后启动所有的变形虫（genes 文件夹下所有的变形虫实现都会被启动）：

```
pc start
```
然后程序会问你一些问题，你根据实际情形来进行配置。
* Storage Path： docker volume 所挂载的文件路径。推荐用默认
* Memory in GB： 需要为容器预留的内存，推荐 2G，我这里设置的 1G
* 域名：这个域名是为了发布变形虫之后，提供给外界使用的，可以直接用 ip 地址
* Join the Planaria network：如果你想运行一个私有的节点，选择 No，如果你想公开协议或者公开节点，选择 Yes
* Port: 变形虫的对外端口，默认 3000
* BITCOIN_USER： BSV 节点 rpc 的用户名，上文有写
* BITCOIN_PASSWORD： BSV 节点 json-rpc 的密码，上文有写

上面的配置在 Planaria 根目录下的.env 中可以随时修改。然后就会看到代码开始启动两个 docker 容器 (interPlanaria/Planarium 和 interPlanaria/Planaria）。之后就会看到容器启动成功。但是容器启动成功并不等于变形虫已经启动成功，需要通过日志来查看。容器的日志可以通过 docker 命令来查看，但这里我们有 PC 工具，不用和 docker 打交道，直接执行

```
pc logs write

pc logs read
```
write 指的是 Planaria 容器的日志，read 指的是 Planarium 的容器，如果容器里发生任何错误或异常，都可以从日志中获取到信息。如果容器内部没有异常，正常启动，就可以看到容器对外开放了 3000 端口。此时从外部浏览器访问服务器的 3000 端口（http://XXX.XXX.XXX.XXX:3000/），就可以显示如下的画面：


有几个 Gene，就有几个状态机显示在这里，点击 query 进去，就会发现和以下公开的 genesis 数据库一样的 bitquery 查询界面：

[genesis 的 query 界面](https://genesis.bitdb.network/query/1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN)

在自己搭建的 query 中执行默认查询语句，能正确返回结果，说明已经搭建完成了。


然后就可以愉快地使用bitquery玩耍链上数据了。
