# Overview

ABCI is the interface between Tendermint (a state-machine replication engine)and your application (the actual state machine). It consists of a set of *methods*, where each method has a corresponding `Request` and `Response` message type. Tendermint calls the ABCI methods on the ABCI application by sending the `Request*` messages and receiving the `Response*` messages in return.

ABCI是Tendermint(一个状态机复制引擎)和你的应用(实际的状态机)之间的接口。它包含了一个方法集合，每一个方法都有一个相关的`Request` and `Response`的信息类型。Tendermint通过发送`Request` 消息并接收`Response`消息来调用ABCI应用程序上的ABCI方法。

All message types are defined in a [protobuf file](https://github.com/tendermint/tendermint/blob/develop/abci/types/types.proto).
This allows Tendermint to run applications written in any programming language.

所有的信息类型被定义在一个protobuf文件(`types.proto`)中。这允许Tendermint可以运行用任何编程语言编写的应用程序。

This specification is split as follows:

规范被分为以下几种

- [Methods and Types](./abci.md) - complete details on all ABCI methods and
  message types
- [Applications](./apps.md) - how to manage ABCI application state and other
  details about building ABCI applications
- [Client and Server](./client-server.md) - for those looking to implement their
  own ABCI application servers

