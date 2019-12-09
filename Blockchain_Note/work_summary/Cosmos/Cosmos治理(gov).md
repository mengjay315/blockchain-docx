## Cosmos-sdk-治理(gov)模块

这是genesis.go中 `gov`模块的初始值

```json
"gov": {
      "starting_proposal_id": "1",
      "deposits": null,
      "votes": null,
      "proposals": null,
      "deposit_params": {
        "min_deposit": [
          {
            "denom": "stake",
            "amount": "10000000"
          }
        ],
        "max_deposit_period": "172800000000000"
      },
      "voting_params": {
        "voting_period": "172800000000000"
      },
      "tally_params": {
        "quorum": "0.334000000000000000",
        "threshold": "0.500000000000000000",
        "veto": "0.334000000000000000"
      }
    },
```

---------

## 参与链上治理

-------

#### 链上治理入门

https://hub.cosmos.network/zh/delegator-guide-cli.html

Cosmos Hub有一个内建的治理系统，该系统允许抵押通证的持有人参与提案投票。系统现在支持3种提案类型：

- `Text Proposals`: 这是最基本的一种提案类型，通常用于获得大家对某个网络治理意见的观点。
- `Parameter Proposals`: 这种提案通常用于改变网络参数的设定。
- `Software Upgrade Proposal`: 这个提案用于升级Hub的软件。

任何Atom通证的持有人都能够提交一个提案。为了让一个提案获准公开投票，提议人必须要抵押一定量的通证 `deposit`，且抵押值必须大于 `minDeposit` 参数设定值. 提案的抵押不需要提案人一次全部交付。如果早期提案人交付的 `deposit` 不足，那么提案进入 `deposit_period` 状态。 此后，任何通证持有人可以通过 `depositTx` 交易增加对提案的抵押。

当`deposit` 达到 `minDeposit`，提案进入2周的 `voting_period` 。 任何**抵押了通证**的持有人都可以参与对这个提案的投票。投票的选项有`Yes`, `No`, `NoWithVeto` 和 `Abstain`。投票的权重取决于投票人所抵押的通证数量。如果通证持有人不投票，那么委托人将继承其委托的验证人的投票选项。当然，委托人也可以自己投出与所委托验证人不同的票。

当投票期结束后，获得50%（不包括投`Abstain`票）以上 `Yes` 投票权重且少于33.33% 的`NoWithVeto`（不包括投`Abstain`票）提案将被接受，

### [译]针对Cosmos Hub验证人的风险控制方案

[https://medium.com/@kidinamoto/%E9%92%88%E5%AF%B9cosmos-hub%E9%AA%8C%E8%AF%81%E4%BA%BA%E7%9A%84%E9%A3%8E%E9%99%A9%E6%8E%A7%E5%88%B6%E6%96%B9%E6%A1%88-2d6c648d3970](https://medium.com/@kidinamoto/针对cosmos-hub验证人的风险控制方案-2d6c648d3970)