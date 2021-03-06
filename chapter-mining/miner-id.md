# MinerID

矿池是比特币系统中非常重要且核心的基础设施。

真正保护比特币账本安全靠的是矿工（这里的矿工指的是矿池，就是具有交易打包权的服务器）对网络持续的监测，来拒绝掉对网络的攻击。保护账本的动作其实很简单，就是使用 invalidateblock 的方式，把试图攻击的区块给拒绝掉。在这种情况下，矿工用自己的算力投票，来保护比特币真正的账本。

在最早比特币出来的时候，存在 “经过六个区块确认后就会比较安全” 的说法。这也是为什么很多人在使用钱包、交易所转账、入账时都很在意确认数的原因。“确认”是指你的交易已经被某一个矿工打包在区块里，写在了这个矿工认为，最有可能成为最长链其中一部分的某个账本中。

六个确认数则是由于在区块链上随时会出现可能的分叉。那一旦出现分叉是否会影响用户资金的安全呢？在这个关键问题上存在着不少误解，实际上，在比特币领域，区块并非一个重要核心理念，反之交易才是最重要的因素：因为每一个交易都经由交易发送方的数字签名，它是一个合法的数字证明，且是依靠交易来保证的，而不是依靠区块。只要你的交易被任何矿工打包，这笔交易就写入了任何可能的账本中。

矿工在不同交易版本选择上发挥巨大作用，他们的力量来自于每位矿工都独立地验证自己收到的交易信息，并查看它是否合法，同时每位矿工对其他矿工广播自己收到的交易——在这个流程背后，其实存在着经济驱动，思考下这个矿工需要去广播交易的驱动，因为当他将交易更快送达至其他矿工时，他打包的区块也可以更高效传达到了其他矿工手里，这样做出于能够降低矿工自身的孤块率的经济驱动。

对于矿工来说，由于他们被驱动着与其他矿工保持一种竞争与合作的关系，在这种情况下，这些矿工收益可以得到更大提升，这是另一种经济驱动。此时我们假设突然出现了一个双花交易，在矿工看来，他会把这个交易和其它的版本，就是其他矿工手里收到的交易进行对比，看其是否会被其它矿池所接受。如果某一个交易有 99% 的算力都认可他的话，我们可以认为这个交易在下一个区块进入账本的可能性是极高的，是有 99% 的可能；如果某一个交易在网络上面并没有收到有双花的通知的话，则可以认为这个交易应该是能够被写入账本的，这就是我们讲的零确认交易概念。

所以，零确认交易并不是指这个网络上不能有双花，这是很重要的一点。很多做应用的开发者认为，“真正的比特币网络交易就应该是拒绝双花，不能允许它存在”，其实这并不必要，关键只在于，需要收款方能够确定地知道，自己收到的交易是否不存在双花的可能性，就是完成任务了。

如果存在双花的可能性的话，只有可能是生成这个交易的人在作弊导致的这个原因，因为只有他能够签名出另外一个版本的交易，所以从开发者的视角来看这个问题的话：

第一，把交易作为核心的原则进行控制，而不是看区块确认，区块确认其实不重要。

第二，在收到交易以后，应该尽快把这个交易和其他所有的矿工进行交叉的确认，确认这个网络上是否存在另外一个版本的交易。尤其是第二点，很多开发者并没有做到。现在开发者都还依赖于广播这么一个概念，稍后我会针对广播背后存在的问题进行说明。

如果某一个交易在网络上没有发现同时存在的双花交易（这是一个前提），但是 10 分钟以后我们发现出了一个新的块，这个块出现了一个双花交易，把原先的交易给覆盖了，这就是大家所说的矿工双花攻击网络。由某恶意矿工打包了一个区块，这个区块双花了用户的币。但这个双花过程是可以被其他矿工检测到的，因为每一个原先的交易，矿工要想接受它，并和其它所有的矿工进行确认以后（为了防止自己被欺骗），当前矿工是知道这个最初交易的版本是被所有矿工看到且被他们认可的。但是在网络上突然无征兆地出现了另外一个块包含了一笔双花的交易，使得现在的交易不合法了，那只可能说明比特币网络受到了来自恶意矿工的攻击。我回到刚才的问题，我们矿工用什么样的方式来保护比特币的账本呢？就是通过检测可能出现的双花交易，并且拒绝掉饱含着不应被其它矿工认可的交易版本所在的那个区块来完成的。

在这时候网络会不可避免出现分叉，因为我们作为诚实的矿工，必须坚守自己认为最真实的账本，所以我们不会允许像别人宣传的那样说——“根据节点代码，你收到那个速度最快的来自攻击者的区块，你应该把自己的算力切到它那边去，去跟跟随着攻击者的区块走”，这是一个很大的概念误区，如果对对挖矿的概念不太了解的人，就可能会误以为双花比特币 SV 的成本很低，因为你只需要用全网百分之一点几的算力，就可以发动攻击，就可以带着所有不明真相的 BSV 账本分叉，有权利在上面任意修改账本。

如果是这样的话，攻击者大可以放马过来试一下。我们作为矿工会坚守我们看到的诚实的账本，不管背后有多少算力。我们会忠于坚持直到自己破产为止，所以这并非简单一句话说，我只要快速的打了 6 个块，然后所有其它的账本、交易所全部都跟着我了。

对于用户而言，假设你在网络上看到了两个不同版本的账本，一个账本是诚实的，他拿到的算力暂时比较低，另外一个版本是攻击的账本，上面有很多非法的双花交易，那你应该选择哪个账本，是选择算力更大的账本还是选择一个诚实的账本，这个问题我将留给所有参与的各方，留给所有的交易所，留给所有的应用开发者，他们会有自己的判断。

讲到这里，我返回说明一下前面提到的广播。广播其实是一个魔法，这个魔法给很多人带来了幻觉。我作为一个矿工，我现在正在舞台上跟大家广播，我宣布一下自己收到了什么交易，就像突然间全世界都听到了我这句话，他们在账本上面就如魔法般记下了这个新的转帐交易，然后这个转帐就很神奇的达成了。这是大家原本以为的魔法。

在比特币世界里，其实不存在大家原本所想像的这种广播。传统意义上，每个节点只保持 6-7 个到其它节点的连接，假设你收到了一个交易，你把它转发给下一个节点，你也只能跟 6-7 个其它节点进行交易的传递，它并非一个广播，但现在很多人把它错误理解为广播。

就是说，并不存在一个方法——“你只需网络大吼一声，就能够保证它传达至网络上连入的所有的其它节点而没有错误”，这件事情是非常难达成的。所以，我们在做开发的时候，需要提醒自己不能依靠 P2P 网络广播的方式来传递交易。这种方式，把实际成本转嫁到了接收方手里，对接收方来说首先需要监听并过滤掉当前网络上的每一条信息，然后找出跟自己相关的那条信息，才能最终确认自己收到的钱。

打一个比方，就好像我要转账给栗子小姐，然后我在我的支票上面写上了栗子小姐收，然后把它往地上一丢，跟栗子小姐说我钱转给你了，已经丢出去了，什么时候收到，看有谁捡到这个支票以后能传到你手里。

但这个做法与愿望背道而驰，本质上我们使用比特币是因为它能够更节约成本，不能通过如此低效率的方式来完成交易传递。我应该写完支票直接塞给她。所以这其实大家在了解比特币世界存在的另外一个重大误区——交易是点对点的。

这代表说我需要把发给对方的交易，签完名直接发到对方手里，然后只有接受方才在意自己的钱是否入账，他会很着急的把这个交易转发每一个他能接触到的矿工，让矿工帮忙检测网络上是否存在双花，并把该交易写入矿工账本里，这才是真正的比特币交易过程。

我之前从巴厘岛的 CambrianSV 会议回来，我发现很多开发者并没有真正的意识到如何正确使用比特币网络，这或许是一个大问题。如果每个交易都还是依赖于 P2P 网络广播，然后大家再从另外一个接口，用另一套程序去监听消息的传递，最后才收入交易的话，整个流程效率极低，这种做法在我们将来实现 GB 级别甚至 TB 级别区块的时候，难度和成本也非常高。

所以，从现在开始，所有的开发者可以尝试完全摒弃掉广播交易的这个做法，以后不再有广播的概念。你要把交易发给谁，你就把交易签了字直接发给对方，由收款方向矿工将这个交易进行验证和入账。只有这样，才能够实现比特币的扩容，才能够帮助我们矿工更快的达到 TB 级别的区块，否则所有的基础设施都会坏掉，当然坏掉的时候不要来找我，这不是我搞坏的，是这些基础设施实现方式错了。

目前在大家使用这个钱包的时候，包括我自己的打点钱包，都存在一个问题：当一个用户持续不断的高频使用，产生成千上万个比特币历史使用地址时，随之面临的是巨大的性能问题，即我们需要监听网络，保证如此多的收款地址，每份资金能够安全无误地转出转入和记账，这非常奇怪，我需要去监听、查看丢在地上的支票背后写的收款人是我，然后我想办法把它捡起来。这种做法让交易成本上涨，那我们其实可以预见未来很多钱包将逐渐摒弃这种收款模式——以后所有交易都是点对点的，我们也将提供一系列基础设施，包括 PKI、CA，来协助用户完成点对点、直接的交易。

接下来，我谈谈未来比特币网络如何能够做到这一点。

但直接连接并不是像大家想的那样，我连接上矿工服务器并开一个连接，就可以传递数据了，矿工也自然的帮我转播交易了。不是这样的，因为交易的转播也是有成本的，矿工为什么要免费的帮别人去传播呢？如果你发出的交易是带手续费的，那对于矿工来说他很自然的会接受你这样的交易，因为他为了拿取后面的手续费。但是这里面会有很多的交易，矿工是拿不到手续费的，这时候你就需要一个叫做支付通道的技术，来额外的向矿工付钱。

现在就要说到我自己的矿池产品了。接下来，mempool 会对开发者正式开放支付通道技术，让开发者使用支付通道为自己的交易打包付一些微小的费用。大家不是抱怨说，现在在比特币上传照片、传视频，传各种各样的东西很贵，那在打包不同种类的交易时，mempool 具备了一定的自主权，因此我可以实现更低费率来接受这样的交易，只要你使用支付通道，把这些交易传递给我们，并为这些交易的打包付足够的费用就足够。大家会发现，我们能够提供在市场上极具竞争力的价格，这是其一。

其二，在支付通道下，每一个消息传递全部都是有数字签名的，这都是数字的证据，它和现有 HTTP 和 API 等最大区别就在于，我们可以为我们的数据本身甚至数据传递定价。也就是将来你或许可以付钱给矿工，付一笔小费用，同时告知矿工去某一个公钥的服务器上取回用那个服务器的公钥签名的想要的信息。矿工可以通过传递这个被签名的信息来收取他的服务费。

这一系列是接下来会在比特币 SV 网络上发生的事情。作为开发者我们不能只看眼前，在座的开发者就应该想到接下来会发生什么事情，接下来会出现只以比特币协议作为基础协议的浏览器，不再是 HTTP。接下来会出现用户自身客户端就有钱包可以通过签发交易的方式来传递信息。

在这个基础上，我们才可以去设想、去构筑接下来比特币的应用。为什么我要强调支付通道这么一个技术呢？每一次的付钱我们只要一次交易付费就好了，这又涉及了另外一个开发的误区——大家什么都不想，认为上链就有好处，认为将一切东西放在链上就解决了问题。

其实上链的成本一直存在的，它是一个被称作 “三因素记账” 的账本，在全球几十个矿工的服务器上永远保存着你的信息。但首先对数据安全性的要求高到愿意付出如此昂贵成本，否则并不需要把所有数据全都放在链上。

再举一个大视频云存储的例子，我可以找多个的云存储服务商，我把我的视频分段切片后放在他们的服务器上，并且让他们使用哈希上链的方式来保证他们提供数据的完整性。我可以通过向他们查询哈希的方式获得原始数据。

在这种情况下，用户可以以极低的成本获得高可用性的数据，因为第一有好几个服务商。其二，这里的数据的完整性有一个唯一的标准，你可以举证对方给你的数据是错的，因为这个数据的哈希存在链上，可被证明。

在这种情况下你并不需要特别在意你的数据是不是真的放在了链上。我们要以更高效的方式去使用比特币的账本空间，而这个账本空间的成本永远是存在那的。我们以一个更高效的方法来用它，来降低服务和应用的成本。而不是想着既然上传费用这么便宜，我把东西都丢到链上去就好了，真正适合放在链上的数据往往具备这么几个特征：

1、具有公开的价值。私人数据放在链上是用于存储，但是放在链上的数据绝大部分是用于和其他的商业伙伴进行共享。

2、这部分数据的完整性、真实性的校验是非常高的，他的需求是很强的。你需要证明在某时某地确实存在这样的一份数据，它就具备着很高的价值。

比如说我们之前在 Bitcoin SV 上面出现的一个叫做 RateSV 应用，他把交易所的分钟级别的交易数据传到了链上，作为企业结算、报税的一个参考数据，那么他就很好的使用了这两个特点。很有价值的数据需要公开，需要存在性证明。要从这些方面去想，才能够知道说我们究竟要用区块链去解决一个什么问题，而不是你手里面掌握的锤子到处都是钉子，看到一个脑袋就想敲一下。
