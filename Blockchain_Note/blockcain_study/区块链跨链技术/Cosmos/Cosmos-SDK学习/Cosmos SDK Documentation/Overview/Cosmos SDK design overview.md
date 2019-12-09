# Cosmos SDK design overview

https://cosmos.network/docs/intro/sdk-design.html#baseapp

It comes with a `multistore` to persist data and a `router` to handle transactions.

cosmos-sdk使用 `multistore`存储数据，使用 `router`处理交易。

Here is a simplified view of how transactions are handled by an application built on top of the Cosmos SDK when transferred from Tendermint via `DeliverTx`:

1. Decode `transactions` received from the Tendermint consensus engine (remember that Tendermint only deals with `[]bytes`).
2. Extract `messages` from `transactions` and do basic sanity checks.
3. Route each message to the appropriate module so that it can be processed.
4. Commit state changes.

The application also enables you to generate transactions, encode them and pass them to the underlying Tendermint engine to broadcast them.

-----

## Multistore（不熟悉）

------

The Cosmos SDK provides a multistore for persisting state. The multistore allows developers to declare any number of [`KVStores`](https://github.com/blocklayerhq/chainkit). These `KVStores` only accept the `[]byte` type as value and therefore any custom structure needs to be marshalled using [go-amino](https://github.com/tendermint/go-amino) before being stored.

The multistore abstraction is used to divide the state in distinct compartments, each managed by its own module. 

多存储抽象化被用来划分不同组件的状态，**每一个都被它们自己的模块管理**。

-------



