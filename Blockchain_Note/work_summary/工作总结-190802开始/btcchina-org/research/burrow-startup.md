### 准备环境
#### 下载
burrow的开发分支为develop，所以下载后切换到develop分支
```
git clone https://github.com/hyperledger/burrow.git
git checkout develop
```
#### 编译
由于代码中使用了SIGUSR1，windows平台编译失败，mac/linux平台：
```
make build
```
如果由于GFW原因下载依赖失败，则在makefile中加入下面一行
```
export GOPROXY=https://goproxy.io
```
#### 安装
```
make install
```
安装在$GOPATH/bin目录下  
注意：由于使用的install命令安装不是用的cp，mac的install命令没有-T选项，所以mac下要编辑makefile，删除install命令的-T选项  

### 单节点  
burrow的配置文件分为两类，创世文件和主配置文件，创世文件可单独存放也可放在主配置文件中，主配置文件主要是tendermint相关配置，创世文件主要是初始区块链参数，[更多创世文件相关](https://github.com/hyperledger/burrow/blob/develop/docs/genesis.md)，[权限相关](https://github.com/hyperledger/burrow/blob/develop/docs/permissions.md)  
- burrow spec 生成创世模板文件  
- burrow configure 生成创世文件和主配置文件

#### 生成配置文件
```
burrow spec -p1 -f1 | burrow configure -s- > burrow.toml
```
上面的命令等价于
```
burrow spec --participant-accounts=1 --full-accounts=1 > genesis-spec.json
burrow configure --genesis-spec=genesis-spec.json > burrow.toml
```
#### 启动
```
# 指定GenesisDoc中验证者的索引
burrow start --validator=0
# 或者指定验证者的地址
burrow start --address=BE584820DC904A55449D7EB0C97607B40224B96E
```
指定验证者是用来本节点签名的

#### 查看状态
```
curl -s 127.0.0.1:26658/status
```
### 多验证节点
#### 生成配置文件
```
 rm -rf .burrow* .keys*
 burrow spec -f2 | burrow configure -s- --pool
```
以上分别是清理环境和生成具有两个验证者的配置文件，将两个配置文件的Logging的Trace设为true

#### 启动网络
##### 启动第一个节点
```
burrow start --config=burrow000.toml
```
可以在burrow000.log中看到`Blockpool has no peers`,因为验证者节点达不到2/3

##### 启动第二个节点
```
burrow start --config=burrow001.toml
```
此时可以在log中看到`Sending vote message`和`Finalizing commit of block with 0 txs`

##### 查看共识状态
```
curl -s 127.0.0.1:26758/consensus
```

### 发送交易  
burrow开发了一个部署系统，能够解释yaml文件，执行一系列任务，所能执行的任务见[文档](https://godoc.org/github.com/hyperledger/burrow/deploy/def)  

#### 创建部署脚本

```
jobs:
- name: sendTxTest1
  send:
      destination: B8CE081FE89374E9FEC92BCBCAFF5BF90306B497
      amount: 42
```
name: 任务名，自定义  
send: 任务类型  
destination: 目的地址，此处为validator1的地址，在burrow001.toml最顶部可看到  
amount: 金额  

#### 发送

```
burrow deploy --chain 127.0.0.1:10997 --address 8DF1C7566299FFE4C6FC6B1B88F0AD41B17B828C deploy.yaml
```
chain: 指定GRPC地址，端口在配置文件中可找到，如果是默认端口(10997)则不需要  
address: 签名地址，此处是validator0的地址，在burrow000.toml最顶部可看到  

执行成功后会有类似如下输出

```
*****Executing Job*****

Job Name                                    => defaultAddr


*****Executing Job*****

Job Name                                    => sendTxTest1


Transaction Hash                            => 41E0C13D1515F83E6FFDC5032C60682BE1F5B19A
Writing [deploy.output.json] to current directory
```
返回结果保存在deploy.output.json

#### 查看账户

```
$ curl -s 127.0.0.1:26758/accounts
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "BlockHeight": 21,
    "Accounts": [
      {
        "Address": "0000000000000000000000000000000000000000",
        "PublicKey": {
          "CurveType": "",
          "PublicKey": ""
        },
        "Balance": 1337,
        "EVMCode": "",
        "Permissions": {
          "Base": {
            "Perms": "send | call | createContract | createAccount | bond | name | proposal | input | batch | hasBase | hasRole",
            "SetBit": "root | send | call | createContract | createAccount | bond | name | proposal | input | batch | hasBase | setBase | unsetBase | setGlobal | hasRole | addRole | removeRole"
          }
        }
      },
      {
        "Address": "8DF1C7566299FFE4C6FC6B1B88F0AD41B17B828C",
        "PublicKey": {
          "CurveType": "ed25519",
          "PublicKey": "27B4EC5714202A54EB24A21360081B9FE1B0DAF3E25523B31265CB0BE7E8C86A"
        },
        "Sequence": 1,
        "Balance": 99999999999957,
        "EVMCode": "",
        "Permissions": {
          "Base": {
            "Perms": "root | send | call | createContract | createAccount | bond | name | proposal | input | batch | hasBase | setBase | unsetBase | setGlobal | hasRole | addRole | removeRole",
            "SetBit": "root | send | call | createContract | createAccount | bond | name | proposal | input | batch | hasBase | setBase | unsetBase | setGlobal | hasRole | addRole | removeRole"
          }
        }
      },
      {
        "Address": "B8CE081FE89374E9FEC92BCBCAFF5BF90306B497",
        "PublicKey": {
          "CurveType": "",
          "PublicKey": ""
        },
        "Balance": 100000000000041,
        "EVMCode": "",
        "Permissions": {
          "Base": {
            "Perms": "root | send | call | createContract | createAccount | bond | name | proposal | input | batch | hasBase | setBase | unsetBase | setGlobal | hasRole | addRole | removeRole",
            "SetBit": "root | send | call | createContract | createAccount | bond | name | proposal | input | batch | hasBase | setBase | unsetBase | setGlobal | hasRole | addRole | removeRole"
          }
        }
      }
    ]
  }
}
```
### 增加验证者节点
- 清理当前环境

```
pkill burrow
rm -rf * ./burrow* ./keys
```
- 启动一个具有两个验证者的网络

```
burrow spec -f2 | burrow configure -s- --pool --separate-genesis-doc=genesis.json
burrow start --config=burrow000.toml &
burrow start --config=burrow001.toml &
```
- 生成新节点的配置文件

```
burrow spec -v1 | burrow configure -s- --json > burrow002.json
```
- 记录新节点的公钥，然后删除`burrow002.json`中的`GenesisDoc`  
因为该节点加入已存在的网络，所以不需要GenesisDoc，但是要指定持久节点地址和创世文件
- 从`burrow00x.toml`取出`PERSISTENT_PEER`，复制到`burrow002.json`
- 修改端口和RPC配置  

```
RPC.Info.Enabled=false
RPC.GRPC.Enabled=false
Tendermint.ListenPort="25565"
```
以上4个步骤也可以用如下命令替代

```
$ PERSISTENT_PEER=$(cat burrow000.toml | grep PersistentPeers | cut -d \" -f2)
$ burrow spec -v1 | burrow configure -s- --json > burrow-new.json
$ NEW_VALIDATOR=$(jq -r '.GenesisDoc.Accounts[0].PublicKey.PublicKey' burrow-new.json)
$ jq 'del(.GenesisDoc)' burrow-new.json | jq ".Tendermint.PersistentPeers=\"$PERSISTENT_PEER\"" | jq '.RPC.Info.Enabled=false' | jq '.RPC.GRPC.Enabled=false' | jq '.Tendermint.ListenPort="25565"' > burrow002.json
```
- 部署脚本

```
jobs:
- name: AddValidator
  update-account:
    # 把NEW_VALIDATOR替换为新节点的公钥
    target: NEW_VALIDATOR
    power: 232322
```
如果没有记录公钥，可通过下面命令取得

```
burrow keys pub --name Validator_0
```
- 执行

```
burrow deploy -c 127.0.0.1:10997 --mempool-signing=true --address=2398A53368ED3ADD09DF2B23214D7AAAC9B7068C deploy.yaml
```
`address`是第一个validator的地址  
- 查看共识状态

```
curl -s 127.0.0.1:26758/consensus
```
可以看到新的validator，可能需要等待几分钟

- 启动新的validator

```
burrow start --config=burrow002.json --genesis=genesis.json
```
### 部署智能合约
- 清理环境
- 启动节点

```
burrow spec --full-accounts 1 | burrow configure --genesis-spec=- --separate-genesis-doc=genesis.json > burrow.toml
burrow start -v0
```

- 安装[solc](https://solidity.readthedocs.io/en/v0.4.21/installing-solidity.html#binary-packages)
- 智能合约`simplestorage.sol`

```
contract simplestorage {
    uint public storedData;

    constructor(uint initVal) public {
        storedData = initVal;
    }

    function set(uint value) public {
        storedData = value;
    }

    function get() public view returns (uint value) {
        return storedData;
    }
}
```

- 部署脚本`deploy.yaml`

```
jobs:

- name: valueToSet
  set:
    val: 10

- name: simplestorage
  deploy:
    contract: simplestorage.sol
    data:
    - 0

- name: setStorage
  call:
    destination: $simplestorage
    function: set
    data:
    - $valueToSet

- name: queryStorage
  query-contract:
    destination: $simplestorage
    function: get

- name: assertStorage
  assert:
    key: $queryStorage
    relation: eq
    val: $valueToSet
```

- 执行部署

```
# 取出签名用的账户地址
$ jq -r '.Accounts[0].Address' genesis.json
53E211FC5E260B8E4AA08A06189D97498683A5FB
$ burrow deploy --address 53E211FC5E260B8E4AA08A06189D97498683A5FB deploy.yaml
```

### http
burrow提供的http api可通过一下命令得到

```
$ curl 127.0.0.1:26658
Available endpoints:
//localhost:26658/account_stats
//localhost:26658/accounts
//localhost:26658/chain_id
//localhost:26658/consensus
//localhost:26658/genesis
//localhost:26658/network
//localhost:26658/validators

Endpoints that require arguments:
//localhost:26658/account?address=_
//localhost:26658/account_human?address=_
//localhost:26658/block?height=_
//localhost:26658/blocks?minHeight=_&maxHeight=_
//localhost:26658/dump_storage?address=_
//localhost:26658/name?name=_
//localhost:26658/names?regex=_
//localhost:26658/status?block_time_within=_&block_seen_time_within=_
//localhost:26658/storage?address=_&key=_
//localhost:26658/unconfirmed_txs?maxTxs=_
```
可以通过curl或[httpie](https://httpie.org/)工具调用

### GRPC
GRPC有两种方法可以使用，一种是burrow提供的deploy系统，一种是[burrow.js](https://github.com/monax/bosmarmot/tree/develop/burrow.js), 使用develop分支  
示例代码见 [basic app](https://github.com/monax/bosmarmot/blob/develop/example/basic-app/app.js)

