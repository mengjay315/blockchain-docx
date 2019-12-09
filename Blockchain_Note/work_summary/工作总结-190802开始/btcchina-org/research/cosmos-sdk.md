cosmos

#### 每个ABCI接口介绍
https://github.com/tendermint/tendermint/blob/master/docs/spec/abci/abci.md

#### 启动流程
```
cosmos==>cosmos: main
cosmos==>cosmos: execute(startCmd)
cosmos==>tendermint: NewNode
tendermint==>tendermint: InitChain
tendermint==>cosmos: initChainer
cosmos==>tendermint: Start
```

#### 关于经济模型  
https://github.com/cosmos/gaia/blob/master/docs/translations/cn/validators/validator-faq.md
https://mp.weixin.qq.com/s/jsw8M35E28via5Qoix85rA
https://zhuanlan.zhihu.com/p/52924338
https://blog.cosmos.network/introducing-the-hard-spoon-4a9288d3f0df
https://drive.google.com/file/d/1jtyYtx7t1xy9gxEi2T5lXFNd8xUY7bhJ/view

#### cosmos-hub主网启动步骤
- 当前主网cosmos sdk版本是0.34.x, 所以下载cosmos sdk v0.34.7
- 编译安装
```bash
make install
```
- 初始化节点
```bash
gaiad init <名称>
```
- 获取创世文件
```bash
curl https://raw.githubusercontent.com/cosmos/launch/master/genesis.json > $HOME/.gaiad/config/genesis.json
```
创世文件每个字段的含义见  
https://github.com/cosmos/gaia/blob/master/docs/genesis.md  
- 添加种子节点  
从 launch page 获取种子节点，添加到$HOME/.gaiad/config/config.toml
- 启动
```bash
gaiad start
```

#### cosmos sdk开发
##### 1. 从高层视角设计应用
- 应用程序的目标
- 需要的模块(一般在x目录下，如果要自定义模块也放在本目录)
- 应用程序的两部分
  - 状态：所有内容都存储在multistore中，需要定义与自定义模块相关的状态
  - 消息：包含在交易中，触发状态的转变，每个模块定义了一个消息列表和处理方法

##### 2. 整体逻辑
当交易到达tendermint节点时，tendermint通过abci接口传递给应用程序，应用程序对交易解码得到消息，将消息分发到对应的模块，根据handler的逻辑进行处理，如果状态需要更新，handler调用keeper执行更新
```
+---------------------+
|                     |
|     Application     |
|                     |
+--------+---+--------+
         ^   |
         |   | ABCI
         |   v
+--------+---+--------+
|                     |
|                     |
|     Tendermint      |
|                     |
|                     |
+---------------------+
```

所以需要实现app和ABCI

##### 3. baseapp
Cosmos SDK提供了ABCI的实现样板baseapp，app只需嵌入即可  
baseapp职责
- 解码交易
- 提取消息，基本校验
- 路由消息到对应模块
- 提交交易
- 设置BeginBlock/EndBlock
- 初始化状态
- 设置query

##### 4. app开发流程
- app
  - 职责：确定性状态机的核心，主要是聚合各模块，不必实现abci接口，baseapp提供了实现模板，只需要嵌入baseapp
  - 位置：一般在`cmd/app.go`
  - 结构：嵌入baseapp，其他见10
  - 实现：主要是创建app，定义路由，定义初始状态
- 自定义模块的数据类型，一般在`x/<custom module>/types.go`，app一般会有自定义模块
- keeper
  - 职责：一个模块的核心部分，负责与存储交互，引用其他keeper与其他模块交互，keeper的存在主要是为了最小权限原则
  - 位置：一般在`x/<custom module>/keeper.go`
  - 结构：一般包括引用的其他keeper、访问存储空间的key、cosmos的编解码模块
  - 实现：一般就是构造函数、get、set等
- 消息
  - 职责：触发状态转变
  - 位置：一般在`x/<custom module>/msgs.go`
  - 结构：根据应用情况定义
  - 实现：固定的接口，在`types/tx_msg.g`o中定义
    - Route(), Type(): 用于将消息路由至合适的模块
    - ValidateBasic(): 基本的无状态检查
    - GetSignBytes(): 编码消息，大部分编码成排好序的JSON
    - GetSigners(): 签名者
    - 构造函数
- handler
  - 职责：处理特定的消息，本质是个子路由，根据消息类型分发
  - 位置：一般在`x/<custom module>/handler.go`
  - 实现：NewHandler，交给对应的keeper处理
- 查询器
  - 位置：一般在`x/<custom module>/querier.go`
  - 结构：定义几个查询器，每个查询器的输入和响应类型
  - 实现：NewQuerier，按照惯例响应类型都是能JSON化和字符串化的，返回值是JSON编码
- 编解码
  - 职责：官方文档说app创建的任何接口和实现接口的任何结构都需在RegisterCodec声明
  - 位置：一般在`x/<custom module>/codec.go`
  - 实现：RegisterCodec
- CLI  一般包括三部分
  - 查询交互
    - 位置：`x/<custom module>/client/cli/query.go`
    - 职责：为每个查询请求定义cobra.Command
    - 查询路径：custom/模块/查询请求/参数
  - 交易生成
    - 位置：`x/<custom module>/client/cli/tx.go`
    - 职责：为每个交易请求定义cobra.Command
  - module client
    - 位置：`x/<custom module>/client/cli/module_client.go`
    - 职责：导出客户端功能的标准方法，会用于cli的main.go
- REST
  - 位置：`x/nameservice/client/rest/rest.go`
  - 实现
    - 注册路由
    - 定义查询请求处理方法
    - 定义交易请求处理方法
- 模块组合
  - 更新结构
    - 在app结构中添加所有用到模块的key和keeper
  - 更新构造函数
    - 生成每个keeper的storeKey
    - 实例化每个keeper
    - 注册每个模块的handler
    - 注册每个模块的querier
    - 设置initChainer，用来定义应用程序初始状态
    - 定义initChainer
    - MakeCodec，注册所有模块
- 入口
  - daemon
    - 位置：`cmd/<custom module>d/main.go`
    - 实现
      - 大部分代码组合了Tendermint、Cosmos SDK、自定义模块的CLI命令
      - InitCmd：生成创世状态
      - AddGenesisAccountCmd：将账户添加到创世文件中
  - client
    - 位置：`cmd/<custom module>cli/main.go`
    - 实现
      - 结合了Tendermint、Cosmos SDK、自定义模块的CLI命令
      - 之前的 Module Client 用在这里
      - 注册路由
- 构建
- 编译运行

