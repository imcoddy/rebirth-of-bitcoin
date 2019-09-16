# 比特币R-Puzzle签名

R-Puzzle 是 CSW 在 2019 年 Coingeek Toronto 会议上提出的一个新的比特币签名方式。简单地说，就是 把数据的指纹（Hash值）放到区块链上，而不是放数据本身。这样更加安全。

通过使用 R-Puzzles， 有一个应用场景，就是减少使用公钥（进而得到比特币地址），而可以代之用 R-Puzzles，从而可以提高隐私性。R-Puzzles 让大家方便地证明， 你知道某个秘密（或者知识），但是不需要展示秘密本身。你的秘密和知识，还是保护得很好。R-Puzzles 使得用户，可以方便地控制自己数据的访问权限， 方便出卖数据访问权限，转移数据控制、访问权限。

其中一个重要的点是，don't use your public key publicly (不要轻易暴露你的公钥)。

https://www.youtube.com/watch?v=CqqTCsLzbEA