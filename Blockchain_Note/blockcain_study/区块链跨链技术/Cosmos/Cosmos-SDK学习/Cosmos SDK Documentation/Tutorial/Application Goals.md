# Application Goals

The goal of the application you are building is to let users buy names and to set a value these names resolve to. The owner of a given name will be the current highest bidder. In this section, you will learn how these simple requirements translate to application design.

你正在构建的应用程序的目标是让用户购买名称并设置这些名称解析的值。给定名称的所有者将是当前最高的出价者。在本节中，你将了解这些简单的需求如何转化为应用程序设计。

A blockchain application is just a [replicated deterministic state machine](https://en.wikipedia.org/wiki/State_machine_replication). As a developer, you just have to define the state [machine](https://tendermint.com/docs/introduction/introduction.html) (i.e. what the state, a starting state and messages that trigger state transitions), and Tendermint will handle replication over the network for you.

区块链应用程序只是一个[复制的确定性状态机](https://en.wikipedia.org/wiki/State_machine_replication)。作为开发人员，你只需要定义状态机（即状态，启动状态和触发状态转换的消息），[Tendermint](https://tendermint.com/docs/introduction/introduction.html)将为你处理网络复制。


Tendermint is an application-agnostic engine that is responsible for handling the networking and consensuslayers of your blockchain. In practice, this means that Tendermint is reponsible for propagating and ordering transaction bytes.Tendermint Core relies on an eponymous Byzantine-Fault-Tolerant (BFT) algorithmto reach
consensus on the order of transactions. For more on Tendermint, click [here](https://tendermint.com/docs/introduction/introduction.html).

Tendermint是一个与应用程序无关的引擎，它负责处理你的区块链的网络和共识层。特别地，这就意味着Tendermint负责传播和排序交易字节。Tendermint 内核依赖于称为拜占庭容错(BFT)算法在对交易排序时达成共识。关于Tendermint的更多信息，点击[这里](https://tendermint.com/docs/introduction/introduction.html)。

The [Cosmos SDK](https://github.com/cosmos/cosmos-sdk/) is designed to help you build state machines. The SDK is a modular framework, meaning applications are built by aggregating a collection of interoperable modules. Each module contains its own message/transaction processor, while the SDK is responsible for routing each message to its respective module.

SDK被设计帮助你构建`状态机`。SDK是一个模块化框架，意思是通过聚合可互操作模块的集合来构建应用程序。每个模块都包含他们自己的信息或交易的处理器，而SDK是负责把每个信息发送到它们各自的模块。

Here are the modules you will need for the nameservice application:

下面的模块是你的nameservice 应用程序所需要的：

- `auth`: This module defines accounts and fees and gives access to these functionalities to the rest of your application.
- `auth`:该模块定义了帐户和费用，并为你的应用程序的其余部分提供了访问这些功能的权限。

- `bank`: This module enables the application to create and manage tokens and token balances.
- `bank`:此模块使应用程序能够创建和管理token和token余额。

- `staking` : This module enables the application to have validators that people can delegate to.
- `staking` :这个模块使应用程序能够有人们可以委托的验证者。

- `distribution` : This module give a functional way to passively distribute rewards between validators and delegators.
- `distribution` :该模块提供了一种在验证者和委托者之间被动地分配奖励的功能方法。

- `slashing` : This module disincentivizes people with value staked in the network, ie. Validators.
- `slashing` : 该模块使那些在网络中有价值的人不感兴趣，例如，验证者。

- `nameservice`: This module does not exist yet! It will handle the core logic for the `nameservice` application you are building. It is the main piece of software you have to work on to build your application.
- `nameservice`:这个模块还不存在！它将处理你正在构建的`名称服务应用程序`的核心逻辑。它是你构建应用程序时必须使用的主要软件。

Now, take a look at the two main parts of your application: the `state` and the` message types`.

现在，看一下应用程序的两个主要部分：**状态**和**消息类型**。

## State
---------------------------------------------------------------------------------------------------------------------------------------------------------

The state represents your application at a given moment. It tells how much token each account possesses, what are the owners and price of each name, and to what value each name resolves to.

状态代表了在特定时刻的你的应用。它告诉每个帐户拥有多少token，每个名称的所有者和价格，以及每个名称解析的值。

The state of tokens and accounts is defined by the `auth` and `bank` modules, which means you don't have to concern yourself with it for now. What you need to do is define the part of the state that relates specifically to your `nameservice` module.

token和帐户的状态由`auth`和`bank`模块定义，这意味着你现在不必关心它。你需要做的是特别地定义与你的`名称服务`模块相关的状态部分。

In the SDK, everything is stored in one store called the `multistore`. Any number of key/value stores (called [KVStores](https://godoc.org/github.com/cosmos/cosmos-sdk/types#KVStore) in the Cosmos SDK) can be created in this multistore. For this application, we will use one store to map `names` to its respective `whois`, a struct that holds a name's value, owner, and price.

在SDK中，所有内容都存储在一个名为`multistore`的存储中。可以在此`多线程`中创建任意数量的键/值存储（在Cosmos SDK中称为[KVStore](https://godoc.org/github.com/cosmos/cosmos-sdk/types#KVStore)）。对于这个应用程序，我们将使用一个存储将`名称`映射到其各自的`whois`，`whois`是一个包含名称值，所有者和价格的结构体。

## Messages
---------------------------------------------------------------------------------------------------------------------------------------------------------

Messages are contained in transactions. They trigger state transitions. Each module defines a list of messages and how to handle them. Here are the messages you need to implement the desired functionality for your nameservice application:

消息包含在交易中。它们触发状态转换。每个模块都定义了一个消息列表以及如何处理它们。以下是为名称服务应用程序实现所需功能所需的消息：

- `MsgSetName`: This message allows name owners to set a value for a given name.
- `MsgSetName`:此消息允许名称所有者为给定名称设置值。

- `MsgBuyName`: This message allows accounts to buy a name and become its owner.
- `MsgBuyName`: 此消息允许账户购买名称，并成为它的所有者。

    When someone buys a name, they are required to pay the previous owner of the name a price higher than the price the previous owner paid for it. If a name does      not have a previous owner yet, they must burn a `MinPrice` amount.

    当某人购买名称时，他们需要向该名称的先前所有者支付高于先前所有者为其支付的价格。如果名称还没有以前的所有者，则他们必须burn `MinPrice` amount。(**这一句不是很明白**)

When a transaction (included in a block) reaches a Tendermint node, it is passed to the application via the [ABCI](https://github.com/tendermint/tendermint/tree/master/abci) and decoded to get the message. The message is then routed to the appropriate module and handled there according to the logic defined in the `Handler`. If the state needs to be updated, the `Handler` calls the `Keeper` to perform the update. You will learn more about these concepts in the next steps of this tutorial.

当一个交易(包含在区块中的)到达一个Tendermint 节点，它通过ABCI被传递给应用程序，并且解码得到信息。信息然后被发送到恰当的模块中，根据定义在`Handler`的逻辑处理它。如果状态需要被更新，`Handler`回调`Keeper`去执行更新。你将会在这个教程的下一个步骤中了解到更多关于这些的内容。

## Now that you have decided on how your application functions from a high-level perspective, it is time to [start implementing it](https://cosmos.network/docs/tutorial/app-init.html).

现在你已经从高级角度决定了应用程序的运行(functions)方式，现在是时候开始实现它了

