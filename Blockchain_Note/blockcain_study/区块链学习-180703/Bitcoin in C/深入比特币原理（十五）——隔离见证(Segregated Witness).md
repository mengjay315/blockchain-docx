# 深入比特币原理（十五）——隔离见证(Segregated Witness)
【摘要】 在本节开始前，建议先温习一下深入比特币原理（四）与深入比特币原理（五）的内容，以便更好的理解隔离见证的原理隔离见证(Segregated Witness)是什么？ 在比特币中，见证（witness）通常指的是锁定脚本(locking script)和解锁脚本(scriptSig),在之前对于比特币交易的学习中我们知道锁定脚本与解锁脚本都是交易(transcation)的一部分。顾名思义，...

在本节开始前，建议先温习一下[深入比特币原理（四）](https://bbs.huaweicloud.com/blogs/d4c97558190611e89fc57ca23e93a89f)与[深入比特币原理（五）](https://bbs.huaweicloud.com/blogs/78947b351a9711e89fc57ca23e93a89f)的内容，以便更好的理解隔离见证的原理  
  
**隔离见证(Segregated Witness)是什么？**  
在比特币中，**见证（witness）**通常指的是**锁定脚本(locking script)和解锁脚本(scriptSig)**,在之前对于比特币交易的学习中我们知道锁定脚本与解锁脚本都是交易(transcation)的一部分。顾名思义，隔离见证就是指将锁定脚本与解锁脚本中的部分内容从交易中**分离**出来，放入一个**独立的数据结构**中。隔离见证简称**Segwit**  
  
**Segwit如何工作？**  
目前Segwit支持P2PKH与P2SH支付，加入隔离见证的交易取名为**P2WPKH**与**P2WSH**，分别来看一下他们的区别。  
**1.P2WPKH(Pay-to-Witness-Public-Key-Hash)**  
首先回顾下P2PKH的锁定脚本(scriptPubKey)与解锁脚本(scriptSig)内容  
**P2PKH**  
  scriptSig:    <signature> <pubkey>  
  scriptPubKey: OP\_DUP OP\_HASH160 <20-byte hash of Pubkey> OP\_EQUALVERIFY OP\_CHECKSIG  
  
再来看一下P2WPKH的脚本内容  
**P2WPKH**    
  scriptSig:    (empty)  
  scriptPubKey: 0 <20-byte hash of Pubkey>  
  witness:      <signature> <pubkey>  
P2WPKH的锁定脚本较P2PKH要精简不少，第一位的数字0是**版本号**，有了版本号，未来脚本升级就能更容易的**向前兼容**。  
P2WPKH的**解锁脚本为空**，而真正的解锁脚本内容被移到了**原交易之外**的witness部分。  
  
**2.P2WSH(Pay-to-Witness-Script-Hash)**  
**P2SH**  
  scriptSig:    0 <SigA> <SigB> <2 PubkeyA PubkeyB PubkeyC PubkeyD PubkeyE 5 CHECKMULTISIG>  
  scriptPubKey: HASH160 <20-byte hash of redeem script> EQUAL  
**P2WSH**    
  scriptSig:    (empty)  
  scriptPubKey: 0 <32-byte hash of redeem script>  
  witness:      0 <SigA> <SigB> <2 PubkeyA PubkeyB PubkeyC PubkeyD PubkeyE 5 CHECKMULTISIG>  
P2WSH锁定脚本与P2WPKH类似，第一位是版本号，第二位是**赎回脚本(Redeem script)**的Hash值。  
值得注意的是P2WSH锁定脚本中的Hash值是**256位(32字节)**的，是使用**SHA256(pubkey)**计算得到；而P2WPKH中的Hash值是**160位(20字节)**的，是使用**RIPEMD160(SHA256(pubkey))**计算得到的。  
这样做的目的是让钱包可以根据Hash值的长度区分交易使用的是P2WPKH还是P2WSH。（思考一下为什么P2PKH与P2SH不需要做这样的区分？）  
在P2SH交易中常常会有多重签名验证，所以验证信息会占用更多空间，将这些信息移到原交易之外能更大程度的降低交易大小。  
  
**Segwit如何升级?**  
在前面的章节我们讲述过，比特币的升级往往伴随着**硬分叉**或**软分叉**。软分叉影响较小，但要求更严格，需要非常严谨的考虑向前兼容性问题。  
开发者希望隔离见证能通过软分叉进行升级，也就是说新旧客户端可以在同一区块链上共存。于是我们要考虑两种场景：  
（1）付款人的客户端支持隔离见证，而收款人不支持  
（2）付款人的客户端不支持隔离见证，而收款人支持  
对于第一种情况，如果收款人不支持隔离见证，那最终发布的地址将会是普通地址（P2PKH或P2SH），那所有交易按照原有的规则进行即可。  
而对于第二种情况，聪明的社区开发者想出了一个过渡方案，即将**P2WPKH或P2WSH植入P2SH**。

**P2WPKH植入P2SH后，交易信息如下：**

  scriptSig:    0 <20-byte hash of Pubkey>  
  scriptPubKey: HASH160 <20-byte hash of script> EQUAL  
  witness:      <signature> <pubkey>  
此处的脚本Hash值为**RIPEMD160(SHA256(0 <20-byte hash of Pubkey>))**的结果，将该**脚本Hash转换为P2SH地址**，就是一个兼容segwit的地址，不支持隔离见证的客户端可以正常支付比特币给这种P2SH地址。  
而对于支持隔离见证的客户端，仍可以将验证信息放在witness结构中，当然这种过渡方案的交易会较完全形态的方案稍大一些，但比无隔离见证的情况要小。  
  

**P2WSH植入P2SH**  
  witness:      0 <SigA> <SigB> <2 PubkeyA PubkeyB PubkeyC PubkeyD PubkeyE 5 CHECKMULTISIG>  
  scriptSig:    0 <32-byte hash of redeem script>  
  scriptPubKey: HASH160 <20-byte hash of script> EQUAL  
使用的方法和P2WPKH植入P2SH是一样的  
  
**Segwit的地址什么样？**  
在新版本客户端**激活隔离见证后**，会有较长一段时间才会被大多数钱包升级。所以新旧客户端一开始将会以植入P2SH（上一节已描述）的方式互相兼容。  
当隔离见证被大范围接受后，钱包将开始使用一种新的专门针对Segwit原生的地址类型，这种地址将使用**Base32编码**，而不再使用Base58，即全部使用小写字母和数字表达。举例如下：  
Mainnet P2WPKH: bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kv8f3t4  
Mainnet P2WSH: bc1qrp33g0q5c5txsp9arysrx4k6zdkfs4nce4xj0gdcccefvpysxf3qccfmv3  
  
如果你看到以**bc1**开头的地址，就是使用隔离见证地址进行的交易，如以下[https://blockchain.info上的某交易为例](https://blockchain.xn--info-394f1jqz89cf21hdwd764c/)

![image.png](_v_images/20190201101804421_605131100.png "1524536413171594.png")

**Segwit的区块大小有限制吗？**  
**有限制。**  
比特币的区块大小限制为1000000bytes，由于witness数据不包含在这个限制中，为了防止witness数据被滥用，仍然对总的区块大小做了限制。  
引入了一个新概念叫**块重量(Block weight)**  
Block weight = Base size * 3 + Total size  
Base size是**不包含**witness数据的块大小  
Total size是**包含**了witness数据的总大小  
  
隔离见证限制Block weight <= 4000000  
  
**为什么要使用Segwit？**  
**1.交易延展性**  
比特币的验证信息是交易中第三方**唯一**可以修改的数据，移除交易中的验证信息，可防止**交易延展性攻击**。交易延展性攻击通常是客户端发出交易后，第三方通过修改或增加一些内容到验证信息中，改变**交易ID(txid)**，引起客户端误以为交易失败（实际已成功，但不再是原始的txid）。  
交易延展性问题的解决可以大大提升支付通道(payment channels),链式交易(chained transactions),闪电网络(lighting networks)的可操作性。  
鉴于篇幅有限，交易延展性问题详细信息可参考：https://github.com/bitcoin/bips/blob/master/bip-0062.mediawiki  
  
**2.脚本版本化**  
使用隔离见证后，锁定脚本(Locking script)将被加上**版本号**，从而使脚本语言的升级更容易**向前兼容**，这种不造成太大影响的脚本语言修改方式将加快比特币的创新  
  
**3.网络与存储伸缩性**  
交易中验证数据(witness data)占据了不小的比例，特别是一些复杂的交易，如多重签名，支付通道等.在一些情况下可以占据**超过75%**的交易大小。将验证数据放到交易外，可提高比特币的伸缩性。  
节点可以在验证签名后删除验证数据或在简单支付验证时忽略它，验证数据不需要被传输到所有节点或存储在所有节点的硬盘上。  
另外节点在计算块大小时，会忽略验证数据的大小，从而变相增加了单个区块可以包含的交易数量。  
  
**4.签名验证优化**  
隔离验证降低签名验证的算法复杂性，在隔离验证前，哈希的计算次数为O(n2),使用隔离验证后，计算次数降为O(n)，n为签名操作的数量  
  
对于隔离见证技术，仍然在不断发展中，有兴趣的朋友可以继续保持关注，更详细的信息可参考  
BIP141：https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki