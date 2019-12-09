# peer的设计与实现
    peer节点作为fabric中处理交易、存储区块的角色，承担了重要的作用。
## 1.1 CommandLine 解析
### 1.1.1 peer目录结构
peer目录结构十分清晰，只有一个main.go文件，其余文件夹除common、gossip外均为自命令集合，有chaincode、channel、node、version五个，各司其职，供main.go整合使用。子命令文件夹中，与文件夹名称同名的 .go文件为主要源码文件，其余的均为按功能划分的动作命令源码。

具体目录结构如下：
![](_v_images/20190324221528611_180229128.png)

### 1.1.2 第三方包


### 1.1.3 peer命令结构解析
官网关于peer命令的解析		https://hyperledger-fabric.readthedocs.io/en/latest/commands/peercommand.html?highlight=peer


