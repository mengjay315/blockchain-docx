## cosmos-sdk

1. **Initialize validators's and node's configuration files**.(**初始化验证和节点的配置文件**)

   `nsd init moniker --chain-id namechain`

   配置文件和数据的目录：(default "/Users/lianhe/.nsd")

   .nsd/config/app.toml

   ![image-20190924174938148](assets/image-20190924174938148.png)

   一个交易费必须满足这个配置文件中指定的任意面值的最小值，**也就是说配置文件里可以配置多个用着Gas的token**.

2. **Add both accounts, with coins to the genesis file**

   ##### nsd add-genesis-account $(nscli keys show jack -a) 1000nametoken,100000000stake

   ##### nsd add-genesis-account $(nscli keys show alice -a) 1000nametoken,100000000stake

   ![image-20190924174904687](assets/image-20190924174904687.png)

我试着再给这个账户添加2个币种：出现如下错误

![image-20190924175159878](assets/image-20190924175159878.png)

我把币名改为小写：

![image-20190924175358938](assets/image-20190924175358938.png)

不能添加账户在已存在的地址：`5E282CB57C13C5AAB62ACC152561D0EAC35A41AB`,这个地址是什么地址？

--------

删除原来的`./nsd`目录，重新开始操作：

![image-20190924185722183](assets/image-20190924185722183.png)

可以一次添加多个不同的token

![image-20190924185831735](assets/image-20190924185831735.png)

3. 生成创世交易,出现错误

![image-20190924190342048](assets/image-20190924190342048.png)

----------------

再重新开始

![image-20190924191256350](assets/image-20190924191256350.png)

**接受2个参数，收到了3个，**难道只能传两个参数，

---------------

![image-20190924192325936](assets/image-20190924192325936.png)

![image-20190924192247469](assets/image-20190924192247469.png)

**生成创世交易：**

![image-20190924192510723](assets/image-20190924192510723.png)

**币可以配置多个，最后一个必须是 `stake**`，也可以是其他的，可以再genesis.json中指定，属于staking部分

![image-20190926130818008](assets/image-20190926130818008.png)

---------

**nsd start 启动服务，开始产生区块**

**在交易之前，先检查账户是否有足够的余额**

**另起一个终端，查询余额**

![image-20190924194551549](assets/image-20190924194551549.png)

![image-20190924194647515](assets/image-20190924194647515.png)

### Buy your first name using your coins from the genesis file

nscli tx nameservice buy-name jack.id 5nametoken --from jack

![image-20190924195110941](assets/image-20190924195110941.png)

### Set the value for the name you just bought

nscli tx nameservice set-name jack.id 8.8.8.8 --from jack

![image-20190924195442972](assets/image-20190924195442972.png)

### Try out a resolve query against the name you registered

nscli query nameservice resolve jack.id

![image-20190924195616101](assets/image-20190924195616101.png)

### Try out a whois query against the name you just registered

nscli query nameservice whois jack.id

![image-20190924195758490](assets/image-20190924195758490.png)

### Alice buys name from jack

nscli tx nameservice buy-name jack.id 10nametoken --from alice

![image-20190924200052467](assets/image-20190924200052467.png)

jack买时名字：jack.id时，花了 100atoken,

这里alice 从jack那里买这个名字，用了120 btoken, alice买时用的不是atoken代币，而是btoken代币，交易发生了，没有出错，按道理应该用 atoken来买，并且要大于 jack买时的100atoken,

**难道是cosmos内部初始配置的token,除了token的名字不一样，额度是等价的吗？**

**这部分看代码时结合源码仔细看看**

-----------

## Run REST routes

![image-20190924202721094](assets/image-20190924202721094.png)

![image-20190924203034770](assets/image-20190924203034770.png)

![image-20190925081447790](assets/image-20190925081447790.png)

**"sequence"的意思是当前账户下发生了几个交易**

------

```bash
# Buy another name for jack, first create the raw transaction
# NOTE: Be sure to specialize this request for your specific environment, also the "buyer" and "from" should be the same address
curl -XPOST -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show jack -a)'","chain_id":"namechain"},"name":"jack1.id","amount":"5nametoken","buyer":"'$(nscli keys show jack -a)'"}' > unsignedTx.json

# Then sign this transaction
# NOTE: In a real environment the raw transaction should be signed on the client side. Also the sequence needs to be adjusted, depending on what the query of alice's account has shown.
nscli tx sign unsignedTx.json --from jack --offline --chain-id namechain --sequence 1 --account-number 0 > signedTx.json

# And finally broadcast the signed transaction
nscli tx broadcast signedTx.json
# > { "height": "266", "txhash": "C041AF0CE32FBAE5A4DD6545E4B1F2CB786879F75E2D62C79D690DAE163470BC", "logs": [  {   "msg_index": "0",   "success": true,   "log": ""  } ],"gas_wanted":"200000", "gas_used": "41510", "tags": [  {   "key": "action",   "value": "buy_name"  } ]}

# Set the data for that name that jack just bought
# NOTE: Be sure to specialize this request for your specific environment, also the "owner" and "from" should be the same address
$ curl -XPUT -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show jack -a)'","chain_id":"namechain"},"name":"jack1.id","value":"8.8.4.4","owner":"'$(nscli keys show jack -a)'"}' > unsignedTx.json
# > {"check_tx":{"gasWanted":"200000","gasUsed":"1242"},"deliver_tx":{"log":"Msg 0: ","gasWanted":"200000","gasUsed":"1352","tags":[{"key":"YWN0aW9u","value":"c2V0X25hbWU="}]},"hash":"B4DF0105D57380D60524664A2E818428321A0DCA1B6B2F091FB3BEC54D68FAD7","height":"26"}

# Again we need to sign and broadcast
nscli tx sign unsignedTx.json --from jack --offline --chain-id namechain --sequence 2 --account-number 0 > signedTx.json
nscli tx broadcast signedTx.json

# Query the value for the name jack just set
$ curl -s http://localhost:1317/nameservice/names/jack1.id
# 8.8.4.4

# Query whois for the name jack just bought
$ curl -s http://localhost:1317/nameservice/names/jack1.id/whois
# > {"value":"8.8.8.8","owner":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6","price":[{"denom":"STAKE","amount":"10"}]}

# Alice buys name from jack
$ curl -XPOST -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show alice -a)'","chain_id":"namechain"},"name":"jack1.id","amount":"10nametoken","buyer":"'$(nscli keys show alice -a)'"}' > unsignedTx.json

# Again we need to sign and broadcast
# NOTE: The account number has changed to 1 and the sequence is now 2, according to the query of alice's account
nscli tx sign unsignedTx.json --from alice --offline --chain-id namechain --sequence 2 --account-number 1 > signedTx.json
nscli tx broadcast signedTx.json
# > { "height": "1515", "txhash": "C9DCC423E10E7E5E40A549057A4AA060DA6D6A885A394F6ED5C0E40AEE984A77", "logs": [  {   "msg_index": "0",   "success": true,   "log": ""  } ],"gas_wanted": "200000", "gas_used": "42375", "tags": [  {   "key": "action",   "value": "buy_name"  } ]}

// 下面的DELETE方法没通过

# Now, Alice no longer needs the name she bought from jack and hence deletes it
# NOTE: Only the owner can delete the name. Since she is one, she can delete the name she bought from jack
$ curl -XDELETE -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show alice -a)'","chain_id":"namechain"},"name":"jack1.id","owner":"'$(nscli keys show alice -a)'"}' > unsignedTx.json

# And a final time sign and broadcast
# NOTE: The account number is still 1, but the sequence is changed to 3, according to the query of alice's account
nscli tx sign unsignedTx.json --from alice --offline --chain-id namechain --sequence 3 --account-number 1 > signedTx.json
nscli tx broadcast signedTx.json

# Query whois for the name Alice just deleted
$ curl -s http://localhost:1317/nameservice/names/jack1.id/whois
# > {"value":"","owner":"","price":[{"denom":"STAKE","amount":"1"}]}
```

**DELETE方法没有通过**

![image-20190925111318123](assets/image-20190925111318123.png)

![image-20190925111402881](assets/image-20190925111402881.png)

-------------

-------

## 问题：

1. 购买一个name时，命令行有个`amount`参数，是你这次购买名字的币，是一个`string`，**可以传多个币种**

![image-20190925142526676](assets/image-20190925142526676.png)

"amount":"100atoken,100btoken,100ctoken"

代码里有解析"amount"的方法,会把这个string按照","分隔，把每个币取出放在切片中，返回的是coin的切片

![image-20190925142849538](assets/image-20190925142849538.png)

![image-20190925142924880](assets/image-20190925142924880.png)

![image-20190925143012105](assets/image-20190925143012105.png)

-----------

--------

## Gaia 根据事件来查询交易，可以批量查询符合条件的交易

Gaiacli query txs --tags

![image-20191010193926883](assets/image-20191010193926883.png)

下面发币的交易

![image-20191010194009591](assets/image-20191010194009591.png)

根据事件查询所有发新币的交易

![image-20191010194149492](assets/image-20191010194149492.png)

` zchaincli query txs --tags 'message.action:add_new_coin' --page 1 --limit 30`

`key:value`,**key要加上事件类型**

--------

-------

## 基于zchain链的钱包后端接口

### Cosmos-sdk的所有rest-ful接口

https://cosmos.network/rpc/#/ICS20/get_bank_balances__address

----

### 关于cosmos-sdk CLI和RESTFule的讨论

https://github.com/cosmos/cosmos-sdk/issues/324

### Implement Gaia-lite LCD Spec #2113

https://github.com/cosmos/cosmos-sdk/issues/2113

### LCD REST-server API #383

https://github.com/cosmos/cosmos-sdk/issues/383

---------

### 一个封装的库，可以与cosmos-sdk交互

Pure Dart library allowing you to easily create an HD Wallet and create, sign and send Cosmos-SDK transactions.

**官网:** https://pub.dev/packages/sacco

**github:** https://github.com/commercionetwork/sacco.dart

--------

## Cosmos Hub Wallets

[Atomic Wallet](https://atomicwallet.io/) - Android, Linux, macOS, Windows

https://atomicwallet.io/

https://github.com/Atomicwallet

#### Universal Cryptocurrency Wallet

##### Secure, exchange and buy over 300 cryptocurrencies and tokens in a single interface.

我安装了mac系统的钱包客户端

密码：426gmj469!

助记词：forward marine entry case worth arrive school hint piece route cinnamon outside

------------

## tendermint events订阅（Subscribe）机制

-----

``/Users/lianhe/gaomengjie/work/btcchina-org/zchain/app/app.go`

![image-20191014193256049](assets/image-20191014193256049.png)

`/Users/lianhe/gaomengjie/work/btcchina-org/zchain-sdk/types/module/module.go`

![image-20191014193428596](assets/image-20191014193428596.png)



/Users/lianhe/gaomengjie/work/my-chain/blockchain/mengjay315/tendermint/rpc/core/events.go`

--------

-------

## Cosmos-sdk的Staking模块

1. 查看所有的 `validators`

   有一个`validators`列表,可以按**投票权**和**自我委托比率**以及**佣金率**分别进行**进行排序显示列表**。

   ![image-20191015095811369](assets/image-20191015095811369.png)

![image-20191015100501468](assets/image-20191015100501468.png)      



命令行查询指定高度的验证者集合**：

> zchaincli query tendermint-validator-set 45610

```json
{
  "block_height": "45610",
  "validators": [
    {
      "address": "cosmosvalcons1xc0ymv2al3yd6tjrhrkqc0plt753ukfs3ven7z",
      "pub_key": "cosmosvalconspub1zcjduepq43l4f9yx8hng6m4xkdkjfw4pf9jw5fcjlvfn70c3da5jkt6shzpsltldk0",
      "proposer_priority": "0", // 提案优先权
      "voting_power": "100" // 投票权重
    }
  ]
}
```







## -----------------

## 测试网 gaiad start启动问题

```
lianhe@hedeMac-mini ~$ gaiad start
I[2019-10-16|20:34:53.942] starting ABCI with Tendermint                module=main 
panic: invariant broken: supply: total supply invariant
	sum of accounts coins: 1825595304867muon
	supply.Total:          11682352019417muon

	CRITICAL please submit the following transaction:
		 tx crisis invariant-broken supply total-supply

```



---------

curl http://127.0.0.1:1317/minting/annual-provisions 

**mint模块**

annual-provisions ：每年的供应量，这个值是怎么变化的？

{
  "height": "3550",
  "result": "2601652631717303236011839965525.386961307668271250"
}

-------

```
//    "genesis":{
//        "genesisFile": "http://127.0.0.1:8000/genesis.json"
//    },
```

[kill -9 `ps | grep node | grep meteor | awk '{print $1}'`](https://stackoverflow.com/questions/12238382/how-to-stop-meteor)

kill -9 ps grep meteor | awk '{print $1}'
ps -ef | grep meteor

--------

### Github cosmos-sdk的 nft-beta分支有关于**nft模块**的代码

![image-20191018111008669](assets/image-20191018111008669.png)

