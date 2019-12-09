### 单节点测试网络

#### 1. 初始化创世文件
```
gaiad init connor --chain-id=zchain-test
```
connor是节点名字  
zchain-test是该链id  

#### 2. 替换代币符号
默认的代币符号是stake，替换成自己需要的代币符号  
```
sed -i s/stake/zcoin/g .gaiad/config/genesis.json
```

#### 3. 创建秘钥
```
gaiacli keys add peggy
```
peggy是秘钥名字  

#### 4. 查看地址
cosmos有三种地址格式  
https://github.com/cosmos/cosmos-sdk/blob/master/docs/spec/addresses/bech32.md  

```
# 账户地址，默认
$ gaiacli keys show peggy -a --bech acc
cosmos1ms4av9vuhgdc4kkk9xalqla03wmx83s728xm73
# 验证器操作地址
$ gaiacli keys show peggy -a --bech val
cosmosvaloper1ms4av9vuhgdc4kkk9xalqla03wmx83s70njwjz
# 验证器共识地址
$ gaiacli keys show peggy -a --bech cons
cosmosvalcons1ms4av9vuhgdc4kkk9xalqla03wmx83s7mqpj7r
```

#### 5. 将账户加入到创世文件中，并分配两种代币
```
gaiad add-genesis-account $(gaiacli keys show peggy -a) 1000000000zcoin,1000000000shitcoin
```

#### 6. 生成创建验证器的创世交易
```
gaiad gentx --name peggy --amount 100000000zcoin
```
默认创建验证器质押的是stake，这里要改为zcoin  
也可以启动节点后再通过create-validator创建验证器  

#### 7. 收集创世交易
```
gaiad collect-gentxs
```
交易放到了创世文件中  

#### 8. 启动节点
```
gaiad start
```

#### 9. 查看验证节点
```
gaiacli query staking validators --trust-node
gaiacli query tendermint-validator-set --trust-node
gaiad tendermint show-validator
```

#### 10. 查看账户余额
```
gaiacli query account $(gaiacli keys show peggy -a) --trust-node
```

#### 11. 查看收益
```
gaiacli query distr rewards $(gaiacli keys show peggy -a) --trust-node
```

#### 12. 提取收益
```
gaiacli tx distr withdraw-rewards $(gaiacli keys show peggy -a --bech val) --from peggy --chain-id zchain-test
```

#### 13. 查询交易
```
# 按交易哈希查
gaiacli query tx AF8BF2D991B927A252D21110BE48F658896418FA212A8A63923A759575E93345 --trust-node
# 按关键字查
gaiacli query txs --tags 'recipient:cosmos1wupsxp4v0r9ujp2jwp47vttawtse5xsyh38ps5' --trust-node | jq .
```

#### 14. 交易费
上述配置不消耗交易费，cosmos的交易费是跟验证节点相关的，每个验证节点单独配置，交易费可以用任何代币任何费用，前提是只要有验证节点接受。最小gas价格可在gaiad.toml中配置，可配置多种代币。

#### 15. 指定gas价格交易
```
 gaiacli tx send $(gaiacli keys show chicken -a) 10shitcoin --from peggy --chain-id zchain-test --gas auto --gas-prices 0.25shitcoin --gas-adjustment=1.15
```
`--gas auto`: 自动估算需要的gas  
`--gas-prices`: gas价格  
`--gas-adjustment`: 如果估算的gas不够，乘以调整系数  
https://github.com/cosmos/cosmos-sdk/issues/3868  

#### 16. 如何获取gas价格
现在没有工具能获取所有节点的最小gas价格，但是在交易时如果不指定gas价格，可以根据失败日志计算出gas价格
```
gaiacli tx send $(gaiacli keys show chicken -a) 10shitcoin --from peggy --chain-id zchain-test --gas auto --gas-adjustment=1.15
```
交易费=gas价格×需要的gas  
新余额=余额-发送的代币-交易费  
https://medium.com/@Pat_who/gas-fees-gas-price-on-cosmos-network-how-do-you-manage-them-bdf304b15a52
