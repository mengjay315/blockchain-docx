# Methods and Types

## Overview

The ABCI message types are defined in a [protobuf file](https://github.com/tendermint/tendermint/blob/master/abci/types/types.proto).

所有的信息类型被定义在一个protobuf文件(`types.proto`)中。

ABCI methods are split across 3 separate ABCI *connections*:

ABCI方法分为3个独立的ABCI连接：

共识连接，内存池连接，信息连接

- `Consensus Connection`: `InitChain, BeginBlock, DeliverTx, EndBlock, Commit`
- `Mempool Connection`: `CheckTx`
- `Info Connection`: `Info, SetOption, Query`

The `Consensus Connection` is driven by a consensus protocol and is responsible for block execution. 

`共识连接`由共识协议驱动，并且负责区块执行。

The `Mempool Connection` is for validating new transactions, before they're shared or included in a block. 

`内存连接`在交易被分享或包含在一个区块之前，验证一个新的交易。

The `Info Connection` is for initialization and for queries from the user.

`信息连接`用于初始化和来自用户的查询。

Additionally, there is a `Flush` method that is called on every connection, and an `Echo` method that is just for debugging.

此外，有一个`Flush`方法在每次连接时被调用，一个`Echo`方法仅仅在调式时使用。

More details on managing state across connections can be found in the section on [ABCI Applications](https://tendermint.com/docs/spec/abci/apps.html).

有关跨连接管理状态的更多详细信息，请参阅ABCI应用程序部分(./apps.md)

## Errors

Some methods (`Echo, Info, InitChain, BeginBlock, EndBlock, Commit`), don't return errors because an error would indicate a critical failure in the application and there's nothing Tendermint can do. The problem should be addressed and both Tendermint and the application restarted.

