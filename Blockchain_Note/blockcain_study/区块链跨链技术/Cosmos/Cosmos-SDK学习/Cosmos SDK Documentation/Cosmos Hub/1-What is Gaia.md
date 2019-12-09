# What is Gaia?

`gaia` is the name of the Cosmos SDK application for the Cosmos Hub. It comes with 2 main entrypoints:

`gaia`是Cosmos Hub的Cosmos SDK应用程序的名称。它有两个主要入口点：

- gaiad: The Gaia Daemon, runs a full-node of the `gaia` application.
- gaiad:Gaia 后台进程，运行`gaia`应用程序的一个全节点

- gaiacli: The Gaia command-line interface, which enables interaction with a Gaia full-node.
- gaiacli:Gaia 命令行接口，可以与一个Gaia全节点交互。

`gaia` is built on the Cosmos SDK using the following modules:

`gaia`是用下面的模块在Cosmos SDK上构建的。

- `x/auth:` Accounts and signatures. -账户和签名。
- `x/bank`: Token transfers. -Token 转移。
- `x/staking`: Staking logic. -股份逻辑(质押逻辑)。
- `x/mint`: Inflation logic. -通胀逻辑。
- `x/distribution`: Fee distribution logic. -费用分发逻辑
- `x/slashing`: Slashing logic. -惩罚逻辑。
- `x/gov`: Governance logic. -治理逻辑。
- `x/ibc`: Inter-blockchain transfers. -区块链内转移。
- `x/params`: Handles app-level parameters. -处理应用程序级别的参数。

About the Cosmos Hub: The Cosmos Hub is the first Hub to be launched in theCosmos Network.
The role of a Hub is to facilitate transfers between blockchains.If a blockchain connects
to a Hub via IBC, it automatically gains access to allthe other blockchains that are
connected to it. The Cosmos Hub is a publicProof-of-Stake chain. Its staking token is called the Atom.

> 关于Cosmos Hub：Cosmos Hub是第一个在Cosmos Network中启动的Hub。 Hub的作用是促进区块链之间的转移。如果区块链通过IBC连接到Hub，它会自动获得对连接到它的所有其他区块链的访问权限。**Cosmos Hub是一个公开的Proof-of-Stake链**。它的(staking-固定的)token称为Atom。

Next, learn how to [install Gaia](https://cosmos.network/docs/cosmos-hub/installation.html).

