# Join the mainnet

## Setting Up a New Node

First, initialize the node and create the necessary config files:

> gaiad init mon_node

You can edit the `~/.gaiad/config/gaiad.toml` file in order to enable the anti spam mechanism and reject incoming transactions with less than the minimum gas prices:

你可以编辑`gaiad.toml`文件，目的是启用反垃圾邮件机制，并且拒绝到来的低于最低gas价格的交易。

## Genesis & Seeds

### Copy the Genesis File

```bash
mkdir -p $HOME/.gaiad/config
curl https://raw.githubusercontent.com/cosmos/launch/master/genesis.json > $HOME/.gaiad/config/genesis.json
```

### Add Seed Nodes

Your node needs to know how to find peers. You'll need to add healthy seed nodes to `$HOME/.gaiad/config/config.toml`. The [`launch`](https://github.com/cosmos/launch) repo contains links to some seed nodes.

**添加种子节点这里不明白**，

![image-20190812123705846](assets/image-20190812123705846.png)

![image-20190812130726688](/Users/lianhe/Library/Application Support/typora-user-images/image-20190812130726688.png)

是在这里添加，我从https://github.com/cosmos/launch这里复制了种子地址，保存后，终端执行 `gaiad start`命令后，出现错误，连接种子节点失败，

不知道是种子节点失效还是怎么回事，我试了所有的种子节点都出现这个错误，

![image-20190812130850896](/Users/lianhe/Library/Application Support/typora-user-images/image-20190812130850896.png)

![image-20190812130657860](/Users/lianhe/Library/Application Support/typora-user-images/image-20190812130657860.png)

![image-20190812124127232](/Users/lianhe/Library/Application Support/typora-user-images/image-20190812124127232.png)

我试了改为true也是出现连接拒绝的错误。

![image-20190812151710278](/Users/lianhe/Library/Application Support/typora-user-images/image-20190812151710278.png)

错误：

E[2019-08-12|16:32:57.181] Error dialing seed                           module=p2p err="incompatible: Peer is on a different network. Got gaia-13005, expected cosmoshub-2" seed=06b158b29797610476e621f28867cbae926fd1d3@163.172.129.132:26656`

网上的问题及解决方案：

cosmos.network/developers

如果不在config.toml配置文件里添加种子节点，则`gaiad start `	启动正常，没出错，

`gaiad status` 查看状态：

![image-20190812131546846](/Users/lianhe/Library/Application Support/typora-user-images/image-20190812131546846.png)

## A Note on Gas and Fees

> On Cosmos Hub mainnet, the accepted denom is `uatom`, where `1atom = 1.000.000uatom`

Transactions on the Cosmos Hub network need to include a transaction fee in order to be processed. This fee pays for the gas required to run the transaction. The formula is the following:

> fees = ceil(gas * gasPrices)

The `gas` is dependent on the transaction. Different transaction require different amount of `gas`. The `gas` amount for a transaction is calculated as it is being processed, but there is a way to estimate it beforehand by using the `auto` value for the `gas` flag. Of course, this only gives an estimate. You can adjust this estimate with the flag `--gas-adjustment` (default `1.0`) if you want to be sure you provide enough `gas` for the transaction.

The `gasPrice` is the price of each unit of `gas`. Each validator sets a `min-gas-price` value, and will only include transactions that have a `gasPrice` greater than their `min-gas-price`.

The transaction `fees` are the product of `gas` and `gasPrice`. As a user, you have to input 2 out of 3. The higher the `gasPrice`/`fees`, the higher the chance that your transaction will get included in a block.

gas依赖于交易。不同的交易需要不同数量的gas。gas的数量当交易被处理时被计算，但是在处理前这有一个估算的方式，通过 gas flag 使用`auto`值。当然，仅仅是给一个估算值。如果你向确保为一个交易提供足够的`gas`，你可以调整这个估算值，使用标识符 `--gas-adjustment(default 1.0)`，

每一个验证者都可以设置一个 `min-gas-price` 值，并且将会包含gasPrice超过 `min-gas-price`。

gas越多，你的交易被包含进区块的机会就越高。

## Run a Full Node

```bash
gaiad start
```

**有错误：** gaia后台进程没有启动，

> gaojie@admin1103:~/.gaiad/config$ gaiad start
> I[2019-08-08|13:55:15.169] Starting ABCI with Tendermint                module=main 
> ERROR: Error during handshake: Error on replay: Validator set is nil in genesis and still empty after InitChain

Check that everything is running smoothly:

```bash
gaiacli status
```

## Export State

Gaia可以将整个应用程序的状态转存到JSON文件中，json文件可以用来手动分析，也可以被用来作为新网络的的genesis 文件。

导出状态：

```sh
gaiad export > [filename].json
```

你也可以从指定高度导出状态（在区块处理结束的那个高度）：

```bash
gaiad export --height [height] > [filename].json
```

![image-20190812132445513](/Users/lianhe/Library/Application Support/typora-user-images/image-20190812132445513.png)

​	如果你计划从导出的状态开始一个新的网络，导出加上`--for-zero-height`标识符。

```bash
gaiad export --height [height] --for-zero-height > [filename].json
```

## Verify Mainnet

---

。。。



## Upgrade to Validator Node

---

You now have an active full node. What's the next step? You can upgrade your full node to become a Cosmos Validator. The top 100 validators have the ability to propose new blocks to the Cosmos Hub. Continue onto [the Validator Setup](https://cosmos.network/docs/cosmos-hub/validators/validator-setup.html).

你可以更新你的全节点成为一个 Cosmos Validator。前100个验证者有能力提出区块到 Cosmos Hub.