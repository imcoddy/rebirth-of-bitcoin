# BSV Planaria 框架技术总结二 Bitquery

此文是变形虫技术总结的第二篇，阅读此文之前建议先阅读关于变形虫的前两篇文章。

[Bitcoin SV 的开发哲学 —— 变形虫框架](https://zhuanlan.zhihu.com/p/62287840)

[BSV Planaria 框架技术总结一 节点搭建](https://zhuanlan.zhihu.com/p/64697171)

前面的文章说过变形虫是一个持久层框架，通过 planaria 组件爬取区块链上的交易数据，提取并加工所需的数据后将数据存储到 MongoDB 中，然后由 planarium 对外提供接口给应用程序来调取数据。在搭建好数据库之后，我们下一步关注的重点就是如何对数据库进行读写。前文已经介绍过，变形虫与传统数据库最大的区别就是读写分离，不能直接写数据库，只能通过客户端发起链上交易来修改状态机，因此向变形虫写数据本质上就是构造交易，这将在下一篇文章中总结。本文的重点就是如何高效优雅地从变形虫中读取数据。

## Bitquery 简介

planaria 使用一套简单高效的查询语言，称为 Bitquery，类似于 sql 语言，可以将变形虫中的数据进行各种图灵完备的组合和处理，输出各种形态的数据。由于变形虫的开发目的是一个可以满足一切基于比特币链上数据需求的持久层框架，因此必须具备一种专为链上交易数据服务的查询语言，bitquery 也就因此而产生。

bitquery 查询语句本质上是一个 json 对象，之所以是这种形态是为了适应 MongoDB 的查询。这里有必要提及一下，变形虫框架为何内置数据库采用的是 MongoDB 而不是类似 mysql 的关系型数据库。照理说，链上交易数据是充分结构化的，各个交易之间，地址之间有很强的关联性，查询交易更适合使用关系型数据库，这样连表操作更为方便，性能也更好。但是对于变形虫而言，它关注的重点并不是交易本身，而是交易上携带的千变万化，各行各业的杂乱无章的非标准数据，这些数据没有统一的范式，也没有统一的关联性。在这种情形下，Nosql 的优势就会凸显出来，将一个个非标准数据看做一个个” 文件 “，不强制要求文件内容的格式，更加符合变形虫的需求目标。我有一种设想，以后的变形虫甚至可以在内部同时集成关系型和非关系型两种数据库，关系型数据库专门处理关联性高的交易数据，提升关联连表查询效率，而非关系型数据库处理各种应用的非标准数据，提升拓展性和兼容性。两种内置的数据库组件各司其职，将变形虫的性能发挥到极致。

bitquery 示例如下：

![bitquery 示例](https://docs.planaria.network/bitquery.png)

bitquery 基于两种非常强大的技术，MongoDB Query Language（mongoDB 查询语言）和 JQ（一个基于栈操作的图灵完备的 json 处理语句）。查询语句如上图所示分为 3 部分，第一部分是协议的版本号，目前是 3，第二部分是基于 MongoDB 的查询语句，筛选出数据，第三部分是返回结果处理语句，对第二部筛选出来的结果进行整理和包装，处理成应用程序需要的样式。

第二部分查询语句主要基于 MongoDB 的查询语句，但是与原生的 MongoDB 查询略有不同，针对 Planaria 做了一定的适配，选取了部分查询功能，使用 json 对象的各个成员来标记搜索条件。

第三部分处理搜索结果，第二步返回的结果是一个 json，可能包含我们不需要的数据，或者不是我们所希望的数据格式，需要将其整理成我们需要的格式，这时候就需要用到 JQ，变形虫内置 JQ 的处理组件，可以直接帮我们把数据在服务器端就处理好，不需要客户端自行进行数据的处理。

query 字段（q）可以包含下列成员（后文会依次详细说明）：

* find：同 MongoDB 的 find 条件语句。
* aggregate：对应 MongoDB 的聚合语句。
* project：对应 mongoDB 的 project 操作符
* sort：对应 MongoDB 的排序操作符
* limit：对应 MongoDB 的个数限制操作
* skip：对应跳过操作，通过 limit 和 skip 可以实现翻页
* db：选择 db，目前有 c（已确认交易库）和 u（未确认交易库）

response 字段（r）包含一个 f 成员，就是后处理 function，里面是一个 JQ 表达式，用于处理 query 返回的结果。

## Genesis 数据储存格式

目前我运行的是一个全量的 Genesis 数据库，因此在这里详细说明一下返回值的字段含义，如果不知道这些字段的含义，也就无从写 query 语句和 Jq 表达式了。没有节点的同学可以直接访问 Unwriter 的公开 endpoint:

[genesis 公开 Endpoint](https://genesis.bitdb.network/query/1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN)

打开后会看到默认的查询语句如下：


```
{
  "v": 3,
  "q": {
    "find": {},
    "limit": 10
  }
}
```

这语句的意思是取最近的十条记录，没有其他条件。然后我们会看到下方有一个巨大的表格，里面填充了各种数据，并且有一些奇怪的表头名称。为了方便描述，我们修改一下查询语句，因为默认查询语句经常会查出一些巨大的交易数据体（二进制图片等数据），在本文有限的空间下没办法展示出来，我们采用如下的查询：


```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "6d02" },
    "limit": 1
  }
}
```

转换成 get 请求如下：

```
GET /q/1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN/ewogICJ2IjogMywKICAicSI6IHsKICAgICJmaW5kIjogeyAib3V0LmgxIjogIjZkMDIiIH0sCiAgICAibGltaXQiOiAxCiAgfQp9 HTTP/1.1
Host: genesis.bitdb.network
key: 1KWqy2WbNpEPC7hwvfJbvXy2vekS2LwGim
Cache-Control: no-cache
Postman-Token: 6445346e-1444-8450-4cb5-088197fe921e
```

这个语句的意思是取出最近的一条 memo.sv（一个 bsv 链上微博）的交易数据，原理我们后文介绍。

可以看到这个查询语句返回了一系列数据，填充在表格中，我们将其 json 源码（从 postman 直接请求）贴下来，如下：

```
{
    "u": [
    ],
    "c": [
        {
            "_id": "5ccfe65cd7faee16774712b0",
            "tx": {
                "h": "15ec948360c54b0864fed10addc9370e7acf70624bae3e46f0db16ba33728e60"
            },
            "in": [
                {
                    "i": 0,
                    "b0": "MEQCIGbAWT8cYm+sHj+/kJI6Ir7HznAZl+uI/Op1w119nyBWAiAmaWlVHAw7F1lPjkremq5YfDyuGr2bMQC7Ft2/uUwor0E=",
                    "b1": "A+8emT7UM3oOjjqstBg4sjn2I4wnIPPwY7ARQ6VkrGue",
                    "str": "3044022066c0593f1c626fac1e3fbf90923a22bec7ce701997eb88fcea75c35d7d9f20560220266969551c0c3b17594f8e4ade9aae587c3cae1abd9b3100bb16ddbfb94c28af41 03ef1e993ed4337a0e8e3aacb41838b239f6238c2720f3f063b01143a564ac6b9e",
                    "e": {
                        "h": "175553b769f70d93b6c3961e8bdefe20ab274f6bbb8bbbcbfbbe205fb347a610",
                        "i": 0,
                        "a": "1Gt8ZMCxkGbkTQfMuPnUZWx8y9ydWfuQA1"
                    },
                    "h0": "3044022066c0593f1c626fac1e3fbf90923a22bec7ce701997eb88fcea75c35d7d9f20560220266969551c0c3b17594f8e4ade9aae587c3cae1abd9b3100bb16ddbfb94c28af41",
                    "h1": "03ef1e993ed4337a0e8e3aacb41838b239f6238c2720f3f063b01143a564ac6b9e"
                }
            ],
            "out": [
                {
                    "i": 0,
                    "b0": {
                        "op": 118
                    },
                    "b1": {
                        "op": 169
                    },
                    "b2": "rjVo4QocscdXU/oca7cRtfhv+8E=",
                    "s2": "\ufffd5h\ufffd\n\u001c\ufffd\ufffdWS\ufffd\u001ck\ufffd\u0011\ufffd\ufffdo\ufffd\ufffd",
                    "b3": {
                        "op": 136
                    },
                    "b4": {
                        "op": 172
                    },
                    "str": "OP_DUP OP_HASH160 ae3568e10a1cb1c75753fa1c6bb711b5f86ffbc1 OP_EQUALVERIFY OP_CHECKSIG",
                    "e": {
                        "v": 26870,
                        "i": 0,
                        "a": "1Gt8ZMCxkGbkTQfMuPnUZWx8y9ydWfuQA1"
                    },
                    "h2": "ae3568e10a1cb1c75753fa1c6bb711b5f86ffbc1"
                },
                {
                    "i": 1,
                    "b0": {
                        "op": 106
                    },
                    "b1": "bQI=",
                    "s1": "m\u0002",
                    "b2": "QSBCaXRjb2luZXIgaW4gdGhlIHN3YXJtIG9mIHNwZWN1bGF0b3JzLi4uCgpodHRwczovL2kuaW1ndXIuY29tLzRNU0xZVW8ubXA0CgojQml0Y29pbg==",
                    "s2": "A Bitcoiner in the swarm of speculators...\n\nhttps://i.imgur.com/4MSLYUo.mp4\n\n#Bitcoin",
                    "str": "OP_RETURN 6d02 4120426974636f696e657220696e2074686520737761726d206f662073706563756c61746f72732e2e2e0a0a68747470733a2f2f692e696d6775722e636f6d2f344d534c59556f2e6d70340a0a23426974636f696e",
                    "e": {
                        "v": 0,
                        "i": 1,
                        "a": "false"
                    },
                    "h1": "6d02",
                    "h2": "4120426974636f696e657220696e2074686520737761726d206f662073706563756c61746f72732e2e2e0a0a68747470733a2f2f692e696d6775722e636f6d2f344d534c59556f2e6d70340a0a23426974636f696e"
                }
            ],
            "blk": {
                "i": 581188,
                "h": "000000000000000005cf5c75cdd7f55b5e772e0d751dd0f4a60076702af5768f",
                "t": 1557128792
            }
        }
    ]
}
```

然后我们根据上面的返回值，介绍一下返回值的各个字段含义。原版的文档请参考如下链接：

[Bitdb Indexer](https://docs.bitdb.network/docs/indexer)

根路径的 u 和 c 代表两个不同的数据库 collection，u 是 unconfirmed tx，未确认交易，存在于内存池中。c 是 confirmed tx，已经确认的交易，存在于区块链中。

然后列表中的每一个对象都是一个交易，里面记载了交易的详情。

对于交易层面而言，其结构如下：


```
{
  "tx": {
    "h": [交易 HASH]
  },
  "blk" {
    "i": [区块高度],
    "h": [区块 HASH],
    "t": [区块时间戳]
  },
  "in": [
    INPUT1,
    INPUT2,
    INPUT3,
    ...
  ],
  "out": [
    OUTPUT1,
    OUTPUT2,
    OUTPUT3,
    ...
  ]
}
```

对多输入多输出的交易，in 和 out 分别是这笔交易输入 input 和输出 output 的列表。

然后我们说输入输出脚本的层面，也就是上面 input 和 output 的每一个具体的对象，建议看这部分之前先了解比特币脚本的工作原理。

这是从上面的 output 中摘取出来的一个输出脚本

```
{  
   "i":0,
   "b0":{  
      "op":118
   },
   "b1":{  
      "op":169
   },
   "b2":"rjVo4QocscdXU/oca7cRtfhv+8E=",
   "s2":"\ufffd5h\ufffd\n\u001c\ufffd\ufffdWS\ufffd\u001ck\ufffd\u0011\ufffd\ufffdo\ufffd\ufffd",
   "b3":{  
      "op":136
   },
   "b4":{  
      "op":172
   },
   "str":"OP_DUP OP_HASH160 ae3568e10a1cb1c75753fa1c6bb711b5f86ffbc1 OP_EQUALVERIFY OP_CHECKSIG",
   "e":{  
      "v":26870,
      "i":0,
      "a":"1Gt8ZMCxkGbkTQfMuPnUZWx8y9ydWfuQA1"
   },
   "h2":"ae3568e10a1cb1c75753fa1c6bb711b5f86ffbc1"
}
```

我们知道，一个标准的 P2PKH（Pay To Pubkey Hash）交易的输出脚本（也称为锁定脚本）如下所示：


```
OP_DUP OP_HASH160 ae3568e10a1cb1c75753fa1c6bb711b5f86ffbc1 OP_EQUALVERIFY OP_CHECKSIG
```

这个脚本可以根据空格拆成 5 个数据或者操作码块，在校验比特币交易时，5 个块依次进栈参与运算。我们按照顺序将其依次标记为 0，1，2，3，4。

然后我们介绍上面的 json 的字段：

* i：指这个输出（或输入）在交易中的标号 index，是第几个输出，0 是第一个
* b0 b1 b2 b3 b4：是上述 5 个块的 base64 编码，如果是操作码，比如 OP_DUP 或者 OP_HASH160，则储存其操作码编号，比如 b0 对应 OP_DUP，其操作码编号是 118，因此 b0 对应的就是 op:118。如果不是操作码，是数据，则储存数据的 Base64 编码。（[比特币操作码编号一栏](https://en.bitcoin.it/wiki/Script#Opcodes)）
* s0 s1 s2 s3 s4：是上述 5 个脚本块的 UTF8 字符串，由于标准脚本只有 s2 是数据，其他都是操作码，因此此处只返回了 s2，操作码不储存成字符串
* h0 h1 h2 h3 h4：是上述 5 个脚本串的十六进制（hex）编码，同样由于标准脚本只有 h2 是数据，是会变化的，因此此处只返回 h2，操作码不储存 hex
* str：脚本原文
* e：edge，边界，对于输入脚本，edge 储存了输入脚本的来源交易，对于输出脚本，edge 储存了这个 utxo 将花向何处
* e（输入脚本中）：h 表示这个输入的来源交易的交易 hash。i 是作为来源的 utxo 在上一笔交易中的 index。a 是来源交易的发起人地址。
* e（输出脚本中）：v 是输出的金额 satoshis，i 是位于输出的位置 index，a 是输出花出去的目标地址（如果该输出已经被花费）


根据上面的解释，我们可以套用相同的模式来解析第二个输出：


```
{  
   "i":1,
   "b0":{  
      "op":106
   },
   "b1":"bQI=",
   "s1":"m\u0002",
   "b2":"QSBCaXRjb2luZXIgaW4gdGhlIHN3YXJtIG9mIHNwZWN1bGF0b3JzLi4uCgpodHRwczovL2kuaW1ndXIuY29tLzRNU0xZVW8ubXA0CgojQml0Y29pbg==",
   "s2":"A Bitcoiner in the swarm of speculators...\n\nhttps://i.imgur.com/4MSLYUo.mp4\n\n#Bitcoin",
   "str":"OP_RETURN 6d02 4120426974636f696e657220696e2074686520737761726d206f662073706563756c61746f72732e2e2e0a0a68747470733a2f2f692e696d6775722e636f6d2f344d534c59556f2e6d70340a0a23426974636f696e",
   "e":{  
      "v":0,
      "i":1,
      "a":"false"
   },
   "h1":"6d02",
   "h2":"4120426974636f696e657220696e2074686520737761726d206f662073706563756c61746f72732e2e2e0a0a68747470733a2f2f692e696d6775722e636f6d2f344d534c59556f2e6d70340a0a23426974636f696e"
}
```

上面的输出是一个 op return 输出，也是变形虫关注的重点。根据上面对字段的解释，我们可以看出，这个输出脚本有三个块，OP_RETURN，6d02，以及后面的数据。


```
OP_RETURN 6d02 4120426974636f696e657220696e2074686520737761726d206f662073706563756c61746f72732e2e2e0a0a68747470733a2f2f692e696d6775722e636f6d2f344d534c59556f2e6d70340a0a23426974636f696e
```

这其实是一个标准的 memo.sv 协议的 op return 格式。开头的 OP_RETURN 是操作码，所以 s0,h0 没有储存，h1 是 6d02 的 hex，6d02 声明了后面的内容属于 memo.sv 协议，再后面的内容就是 hex 化的数据体，关于 bit 协议的声明（6d02 的来源和改良方案），可以参考 bitcom 协议规范，Ifwallet 的 bibodeng 对此专门有讲解：

[Unwritter 发布的 Bitcom 协议是什么？](https://zhuanlan.zhihu.com/p/62712545)

至此，我们详细介绍了变形虫中储存交易的格式，之后就是通过操作 MongoDB 以及 JQ 来查询和处理我们所需要的数据。

## 查询语句

然后我们研究 Bitquery 的 query 部分，查询体。

query 部分的本质是查询 MongoDB，而且由于 Planaria 只可读，在使用它的时候我们只需要关注读取语句（当然如果你要自定制 planaria.js，需要研究 MongoDB 的写入操作，这不在本文的讨论范畴内，以后专门写）。因此我们需要先对 mongoDB 的 find 操作的一些基本概念进行介绍，官方文档如下：

[MongoDB 的 Find 操作](https://docs.mongodb.com/manual/reference/method/db.collection.find/)

find 语句传入一个 json 对象，json 对象中的每一个字段名代表要指定的字段名，字段的值代表搜索条件，多个字段可以并列，相当于 sql 语句中的 and 条件。

这里由于篇幅不够，详细的操作无法全部说明，就挑一些官网的典型示例来说明一下。

1. 匹配相等，index 为 5，姓是 Hopper


```
db.bios.find( { _id: 5 } )

db.bios.find( { "name.last": "Hopper" } )
```

2. 匹配范围或正则表达式


```
db.bios.find(
   { _id: { $in: [ 5, ObjectId("507c35dd8fada716c89d0013") ] } }
)

db.bios.find( { birth: { $gt: new Date('1950-01-01') } } )

db.bios.find(
   { "name.last": { $regex: /^N/ } }
)
```

3. 匹配多个条件（相当于 and）

```
db.bios.find( {
   birth: { $gt: new Date('1920-01-01') },
   death: { $exists: false }
} )
```

4. 匹配多个条件之一（相当于 or）

```
db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
)
```

5. 匹配非（相当于！）

```
db.col.find({"likes":{$ne:50}})
```

6.aggregate 聚合函数（相当于 count）
聚合需要一些聚合操作，这些操作又称为管道 pipeline，MongoDB 将这些管道内的操作（称为 stage）处理完成后，将结果聚合在一起。管道的操作列表及其说明参照下文：

[Aggregation Pipeline Stages](https://docs.mongodb.com/manual/reference/operator/aggregation-pipeline/)

例如，数据源如下

```
{ _id: 1, cust_id: "abc1", ord_date: ISODate("2012-11-02T17:04:11.102Z"), status: "A", amount: 50 }
{ _id: 2, cust_id: "xyz1", ord_date: ISODate("2013-10-01T17:04:11.102Z"), status: "A", amount: 100 }
{ _id: 3, cust_id: "xyz1", ord_date: ISODate("2013-10-12T17:04:11.102Z"), status: "D", amount: 25 }
{ _id: 4, cust_id: "xyz1", ord_date: ISODate("2013-10-11T17:04:11.102Z"), status: "D", amount: 125 }
{ _id: 5, cust_id: "abc1", ord_date: ISODate("2013-11-12T17:04:11.102Z"), status: "A", amount: 25 }
```

使用聚合操作将它们聚合并算出个数


```
db.orders.aggregate([
                     { $match: { status: "A" } },
                     { $group: { _id: "$cust_id", total: { $sum: "$amount" } } },
                     { $sort: { total: -1 } }
                   ])
```

管道的操作是，首先选取 status 为 A 的，然后将它们按照 cust_id 字段进行分组，并统计出每组的 amout 总和，最后按照总和倒序排列。最后的返回结果如下：


```
{ "_id" : "xyz1", "total" : 100 }
{ "_id" : "abc1", "total" : 75 }
```

7.project，或 projection 操作，指定返回的字段，指定返回则设置为 1，指定不返回则设置为 0

例如：

```
const cursor = db
  .collection('inventory')
  .find({
    status: 'A'
  })
  .project({ item: 1, status: 1 });
```

在 inventory 表中选取 status 为 A 的记录，并返回_id，item，和 status，3 个字段。

8.sort 操作，指定排序方式，按照哪个字段排序，正序为 1，倒序为 - 1

例如：


```
db.orders.find().sort( { amount: -1 } )
```

在 orders 表中选取全部，并按照 amount 倒序排列。

9.limit 操作，指定返回的限制个数

按照 query 查找，并限制返回 number 个

```
db.collection.find(<query>).limit(<number>)
```

10.skip 操作，指定跳过的个数 offset

```
function printStudents(pageNumber, nPerPage) {
  print( "Page: " + pageNumber );
  db.students.find()
             .skip( pageNumber > 0 ? ( ( pageNumber - 1 ) * nPerPage ) : 0 )
             .limit( nPerPage )
             .forEach( student => {
               print( student.name );
             } );
}

```

上面是一个配合 limit 使用的翻页逻辑，跳过前 offset 个记录

至此，我们已经简单介绍了 query 部分可能涉及到的一些查询操作，更多高级用法请参阅 MongoDB 的使用规范和文档，本文不赘述。

query 部分示例与详解：


```
{
  "v": 3,
  "q": {
    "find": {
      "$text": {
        "$search": "hello"
      },
      "out.h1": "6d02",
      "out.b2": "hello"
    },
    "skip": 5,
    "limit": 10,
    "sort": { "blk.i": 1 }
  }
}
```

上述的查询条件简单说明：搜索字段中是文本类型且带有 hello 的，而且 out.h1 字段等于 6d02，而且 out.b2 字段等于 hello 的所有记录，跳过前 5 个，限制取 10 条记录，并根据 blk.i（区块高度）正序排列。

## JQ json 后处理语句

在上一步获取到查询结果后，mongoDB 返回的是一个 json，如果我们希望对这个 json 进行一些后处理，就可以使用 bitquery 提供的另一个强大的工具，JQ 后处理。

JQ 是一个轻量级，图灵完备的堆栈式命令行 Json 处理工具。官方网站如下：

[JQ 官网](https://stedolan.github.io/jq/)

bitquery 集成了 JQ，并将其写在请求 json 的第三个部分，r.f 字段。如下所示


```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "534c5000", "out.s3": "GENESIS" },
    "limit": 20,
    "project": { "out.$": 1, "_id": 0 }
  },
  "r": {
    "f": "[.[] | .out[0] | { token_symbol: .s4, token_name: .s5, document_url: .s6} ]"
  }
}
```

f 的一长串值就是 JQ 表达式，上述示例的含义就是取出所有 query 返回值中的 out [0]，然后将其中的 s4 拆出来作为字段 token_symbol 的值，将 s5 拆出来作为字段 document_url 的值，并返回一个新的列表。

查询结果如下所示：


```
{
  "u": [{
    ...
  }],
  "c": [{
    "token_symbol": "TEST",
    "token_name": "TEST",
    "document_url": "bitcoinfiles:b86b4bcbab7cd787b1c893ca101250c8c467dbba4df229b118218bd8a9e85a92"
  }, {
    "token_symbol": "VOTE",
    "token_name": "An Election",
    "document_url": "bitcoinfiles:a90e59ef7ca66b25b6ba98d028198ae222a8229804c4b0b3bc0b1bafe104738a"
  }, {
    "token_symbol": "WuCash",
    "token_name": "Wu Tang Cash",
    "document_url": "http://wu.cash"
  }, {
    "token_symbol": "bb23n",
    "token_name": "bb23",
    "document_url": "bb23n.com"
  }, {
    "token_symbol": "DBOOK01",
    "token_name": "Digital Book Example",
    "document_url": "https://digitalbookexampletokenurl.com"
  }, {
    "token_symbol": ""
    "token_name": ""
    "document_url": ""
  }, {
    "token_symbol": "MTT",
    "token_name": "MyTestToken",
    "document_url": ""
  }]
}
```

jq 的使用方法 unwriter 简单概括如下：

* 所有 jq 表达式都是基于堆栈的，因此从左向右阅读
* 所有的表达式都假设传入一个 json 对象，并最后输出一个对象
* 管道表达式类似于 unix 的管道，它将处理结果从左到右依次传递下去

我们这里受篇幅的限制，简单描述一下一些常用的 jq 语法，英文好的朋友可以直接参照 JQ 官网提供的手册来学习语法，当然也可以参考下面的中文博客，介绍的很详细，并且可以使用里面的 jq play 在线校验表达式。

[jq 常用操作](https://mozillazg.com/2018/01/jq-use-examples-cookbook.html#hidid1)

我们这里列举几个比较常用的语法：

1. 获取 object 下某个字段的值，也可以在其后增加？，表示如果不存在也不会报错

```
.key, .foo.bar, .["key"]

.key?, .foo.bar?, .["key"]?
```

2. 获取所有的值

```
.[]

$ echo '{"url": "mozillazg.com", "name": "mozillazg"}' |jq .[]
"mozillazg.com"
"mozillazg"
```

3. 构造新数组，比如将上面所有的值组成新的数组

```
[.[]]

$ echo '{"url": "mozillazg.com", "name": "mozillazg"}' |jq [.[]]
[
  "mozillazg.com",
  "mozillazg"
]
```

4. 切分数组，选取数组的某一项


```
.[1]  .[0:2]

$ echo '[{"name": "tom"}, {"name": "mozillazg"}, {"name": "jim"}]' |jq .[1]
{
  "name": "mozillazg"
}
$ echo '[{"name": "tom"}, {"name": "mozillazg"}, {"name": "jim"}]' |jq .[0:2]
[
  {
    "name": "tom"
  },
  {
    "name": "mozillazg"
  }
]
```

5. 使用多个 filter，依次执行

如下面依次执行选取 url 的值，选取 name 的值，所以打印了两个值出来

```
,

$ echo '{"url": "mozillazg.com", "name": "mozillazg"}' |jq '.url, .name'
"mozillazg.com"
"mozillazg"
```

6. 管道操作符，类似于 unix 的管道操作，熟悉 linux 操作的一定不陌生

[Pipeline (Unix)](https://en.wikipedia.org/wiki/Pipeline_(Unix))

相当于对管道前后的条件依次过滤，最后输出结果，如下面的语句，就是先选取 json 原文，然后再从中过滤出 url 的值

```
|

$ echo '{"url": "mozillazg.com", "name": "mozillazg"}' |jq '.|.url'
"mozillazg.com"
```

我们这里做一个简单的 bitquery 操作示例（unwriter 的网站给出的），并稍微解释一下，更高级和深入的用法还需要多加研究 jq 的文档。


```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "6d02" },
    "limit": 10
  },
  "r": {
    "f": "[{ block: .blk.i?, timestamp: .blk.t?, content: .out[1].s2 }]"
  }
}
```

上述的 jq 表达式的意思是将每个结果的以下 3 个数据提取出来，构成一个新的对象列表。细心的朋友可能运行上述 bitquery 表达式会报错，我们仔细研究一下，发现确实有问题 (unwriter 大神也会犯错哈)。query 部分返回的直接是个数组，在数组中选择.blk 就会直接报错。这点我们通过变形虫执行上述命令的日志可以看出来错误原因：


```
Query =  { find: { 'out.b1': 'bQI=' }, limit: 2 }
before transform =  [ { _id: 5cd041b74171e816cb23c05b,
    tx:
     { h: 'a1d58e3053bb0aea6c208eca40ba6299fc9e770fb74cfacac4cf90dcc92efbaa' },
    in: [ [Object] ],
    out: [ [Object], [Object] ],
    blk:
     { i: 581226,
       h: '000000000000000008c75b2424d2f4975015c2fc5add0af997e9e332a9b42898',
       t: 1557152175 } },
  { _id: 5cd041b74171e816cb23c17a,
    tx:
     { h: '0982091b7923b82bf3c64fe17a9e72bcb099d72388e30ce1ef1a43efff405628' },
    in: [ [Object] ],
    out: [ [Object], [Object] ],
    blk:
     { i: 581226,
       h: '000000000000000008c75b2424d2f4975015c2fc5add0af997e9e332a9b42898',
       t: 1557152175 } } ]
after transform =  [ { _id: 5cd041b74171e816cb23c05b,
    tx:
     { h: 'a1d58e3053bb0aea6c208eca40ba6299fc9e770fb74cfacac4cf90dcc92efbaa' },
    in: [ [Object] ],
    out: [ [Object], [Object] ],
    blk:
     { i: 581226,
       h: '000000000000000008c75b2424d2f4975015c2fc5add0af997e9e332a9b42898',
       t: 1557152175 } },
  { _id: 5cd041b74171e816cb23c17a,
    tx:
     { h: '0982091b7923b82bf3c64fe17a9e72bcb099d72388e30ce1ef1a43efff405628' },
    in: [ [Object] ],
    out: [ [Object], [Object] ],
    blk:
     { i: 581226,
       h: '000000000000000008c75b2424d2f4975015c2fc5add0af997e9e332a9b42898',
       t: 1557152175 } } ]
running jq
STR =  
error jq SyntaxError: Unexpected end of JSON input
    at JSON.parse (<anonymous>)
    at Socket.child.stdout.on (/app/node_modules/bigjq/index.js:17:29)
    at emitNone (events.js:111:20)
    at Socket.emit (events.js:208:7)
    at endReadableNT (_stream_readable.js:1064:12)
    at _combinedTickCallback (internal/process/next_tick.js:139:11)
    at process._tickCallback (internal/process/next_tick.js:181:9)
query =  { v: 3,
  q: { find: { 'out.b1': 'bQI=' }, limit: 2 },
  r:
   { f: '[{ block: .blk.i?, timestamp: .blk.t?, content: .out[1].s2 }]' } }
response =  { errors: [ 'SyntaxError: Unexpected end of JSON input' ] }

```

可以从日志中看到，这段命令错误的原因在于对列表 [] 进行值选择.blk，那么根据 jq 的命令，正确的写法应该如下所示：


```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "6d02" },
    "limit": 10
  },
  "r": {
    "f": "[.[] | { block: .blk.i?, timestamp: .blk.t?, content: .out[1].s2 }]"
  }
}
```

我把此处修改为首先获取列表中的每一个对象，对每个对象进行管道操作，提取其中的 blk.i，blk.t 和 s2 作为 3 个新的字段的值，组成一个新的对象列表。这样就可以返回正确的答案了（谁认识 Unwriter 麻烦把这个文档中的小错误告知他）。

感兴趣的朋友可以在浏览器测试一下上面前后两个 bitquery，来验证一下是否能正确执行。

正确的结果应该如下所示（篇幅限制这里 limit 3）


```
{
    "u": [],
    "c": [
        {
            "block": 581226,
            "timestamp": 1557152175,
            "content": "capitalism has no answers when it comes to the peoples' interests."
        },
        {
            "block": 581226,
            "timestamp": 1557152175,
            "content": "Facebook just won't stop! They are now planning to have their own coin. http://bit.ly/2DSKT6j\nWill this be a good news or bad for the crypto market? That link is from Cointelegraph."
        },
        {
            "block": 581224,
            "timestamp": 1557150641,
            "content": "Time for another change of topic for my next Monday Memo. Maybe something to do with recreation. Yes, that'll do.\n\nIf you're into hiking like I am, try taking new paths or trails from time to time."
        }
    ]
}
```

## 结语

这些技术文档是我自己学习变形虫协议的时候的学习总结，希望分享给各位开发者。我也希望更多开发者能够了解 BSV 所能实现的强大功能，多多参与进来，共同构建出更多好玩的应用，让区块链技术能够真正落地，不要仅仅停留在炒币和嘴炮上。

写这些文章是会花费很多时间，也需要查阅很多资料，甚至需要自己亲自去一次次的真机实验，但是我相信这并不是无用功，通过写这些技术文档，我对这些技术的了解更为深入，也学习到很多思想和方法，我相信比特币是proof of work，只有真正脚踏实地地work，才能让比特币的价值得以沉淀，让比特币的精神真正发扬下去。欢迎各位技术大神交流指教。
