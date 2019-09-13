# P2IP

## P2IP 简介

中本聪在早期的比特币客户端里提供了一个很有意思的功能：将比特币直接发送到一个 IP 地址上。

> There are two ways to send money.  If the recipient is online, you can enter their IP address and it will connect, get a new public key and send the transaction with comments.  If the recipient is not online, it is possible to send to their Bitcoin address, which is a hash of their public key that they give you.  They'll receive the transaction the next time they connect and get the block it's in.  This method has the disadvantage that no comment information is sent, and a bit of privacy may be lost if the address is used multiple times, but it is a useful alternative if both users can't be online at the same time or the recipient can't receive incoming connections.

* 您的客户联系 IP 地址，以确定他们是否实际运行比特币并接受 IP 交易。如果不是，则不发生任何交易。
* 您的附加信息（“from”，“message”等）与服务器交换。
* 服务器生成一个全新的比特币公钥并将其发送给您的客户端。
* 您的客户端将硬币发送到此公钥。

由于当时的客户端实现并没有提供身份验证功能，因此可能会受到中间人攻击，即任何 “中间人” 都可能在交易期间截获您的比特币。当他们看到有人通过 IP 地址发送比特币付款时，他们假装是实际的收款人并将其比特币地址发回，从而你最终将比特币发送给了错误的人。遗憾的是，不久之后中本聪就消失了，而 Core 开发团队以不安全为由把这个功能从代码中删除。

