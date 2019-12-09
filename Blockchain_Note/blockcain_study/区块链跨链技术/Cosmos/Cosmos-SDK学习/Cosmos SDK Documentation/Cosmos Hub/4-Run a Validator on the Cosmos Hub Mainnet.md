# Run a Validator on the Cosmos Hub Mainnet





## What is a Validator?

[Validators](https://cosmos.network/docs/cosmos-hub/validators/overview.html) are responsible for committing new blocks to the blockchain through voting. A validator's stake is slashed if they become unavailable or sign blocks at the same height. Please read about [Sentry Node Architecture](https://cosmos.network/docs/cosmos-hub/validators/validator-faq.html#how-can-validators-protect-themselves-from-denial-of-service-attacks) to protect your node from DDOS attacks and to ensure high-availability.

验证者通过投票负责提交新的区块到区块链上。如果他们变得不可用或者在相同的高度签名，一个验证者的质押将会被削减。请读关于 [Sentry Node Architecture](https://cosmos.network/docs/cosmos-hub/validators/validator-faq.html#how-can-validators-protect-themselves-from-denial-of-service-attacks) 去避免你的节点遭受 DDOS攻击，并且确保高可用性。

## Create Your Validator

---

Your `cosmosvalconspub` can be used to create a new validator by staking tokens. You can find your validator pubkey by running:

你的 `cosmosvalconspub`通过质押tokens可以被用来创建一个新的验证者。通过下面的命令你可以发现你的**验证者公钥**：

```bash
gaiad tendermint show-validator
```

![image-20190812134609603](/Users/lianhe/Library/Application Support/typora-user-images/image-20190812134609603.png)

> cosmosvalconspub1zcjduepqx82njda7z6y4pjl6tq8ujhel4x7yscrfg9f4lj9vd394t4pf796szzt59as

To create your validator, just use the following command:

```bash
gaiacli tx staking create-validator \
  --amount=1000000uatom \
  --pubkey=$(gaiad tendermint show-validator) \
  --moniker="mock_node" \
  --chain-id=mock_node \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas="auto" \
  --gas-prices="0.025uatom" \
  --from=mock
```

![image-20190812140118583](/Users/lianhe/Library/Application Support/typora-user-images/image-20190812140118583.png)

运行上面的命令前，要先开启个gaia后台进程 `gaiad start`:

![image-20190812141038084](/Users/lianhe/Library/Application Support/typora-user-images/image-20190812141038084.png)

![image-20190812141110421](/Users/lianhe/Library/Application Support/typora-user-images/image-20190812141110421.png)

`ERROR: {"codespace":"sdk","code":1,"message":"failed to load state at height 0; version does not exist (latest height: 0)"}`

