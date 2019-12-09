# Validator FAQ

-----

### What is a validator?













## Becoming a Validator

-----

### How to become a validator?

任何在网络中的参与者都可以通过发送一个 `create-validator`交易来表明他们想成为一个验证者，这个`create-validator`交易必须被下面的参数填写：

- **Validator's** `PubKey`: 和这个Tendermint `PubKey`相关的私钥被用来预签名和预提交。
- **Validator's Address**：应用级别的地址。这个地址被用来识别你的验证者公钥。和这个地址相关的私钥被用来委托、解除绑定、获得奖励，并且参与治理。
- **Validator's name (moniker)**：验证者名字
- **Validator's website (Optional)**：验证者网站
- **Validator's description (Optional)**：验证者描述信息
- **Initial commission rate**: 初始的佣金率。区块奖励的佣金率和向委托者收取的费用（我的理解是分成的比率）。
- **Maximum commission:**** 最高佣金。这个验证者可以获得的最高的佣金率。这个参数在`create-validator`交易被处理之后是不可以改变的。
- **Commission max change rate:**佣金最高变动率。验证者佣金每日最高增幅。这个参数在`create-validator`交易被处理之后是不可以改变的。
- **Minimum self-delegation:**最低自我授权。验证者需要始终绑定的最小的Atoms数量。如果验证者的自我授权资金低于这个限制，他们所有的质押的资金池将会解除绑定。

一旦验证者被创建，Atom持有者可以委托atom给他们，高效地添加资金到他们的资金池。地址的总资金是由委托人托管的Atoms的组合，Atoms由指定他们自己的实体自行结合。

在所有发出声明的候选人中，拥有最高资金的100人被指定为验证者。他们成为了验证者，但是如果一个验证者的所有资金低于前100人的，那么这个验证者将失去他们的验证特权：他们不能参与共识，不再生成奖励。随着时间的推移，最大数量的验证者将会增加，根据下面的时间表（这个时间表可以通过治理被改变）：

- **Year 0:** 100
- **Year 1:** 113
- **Year 2:** 127
- **Year 3:** 144
- **Year 4:** 163
- **Year 5:** 184
- **Year 6:** 208
- **Year 7:** 235
- **Year 8:** 265
- **Year 9:** 300
- **Year 10:** 300

### What are the different states a validator can be in?

验证器可以有哪些不同的状态？

通过一个`create-validator`交易一个验证者被创建后，他们可以有三种状态：

- `in validator set`:在验证者集中。验证者在一个活跃的集合中，并且参与共识。验证者正在挣取奖励，同时由于不端的行为也会被惩罚。
- `jailed`: （监禁状态）。验证者行为不端并且处于监禁，例如，不在验证者集合中。如果监禁是由于离线太久，验证者可以发送一个`unjail`交易，目的是为了再次进入验证者集合。如果监禁是由于双重签名，验证者不能解除监禁。
- `unbonded`:验证者不在活跃集合中，因此不能签名区块。验证者也不会被惩罚，也不会获得任何奖励。他仍然可能委托Atoms给验证者。从无绑定验证者那里取消委托是立即的。

### What is 'self-delegation'? How can I increase my 'self-delegation'?

自我委托是验证者委托给他们自己。通过从验证者的应用程序的应用程序密钥发送`delegate`交易，可以增加此金额。

### Is there a minimum amount of Atoms that must be delegated to be an active (=bonded) validator?

必须委托给验证者的最小Atoms数额？

最小是`1atom`.

### How will delegators choose their validators?

委托人如何选择他们的验证者？

委托人根据他们自己的主观标准选择验证者。预期重要的标准包括：

- **Amount of self-delegated Atoms:** 验证者自我委托给他们自己的数量。拥有更多自我授权Atoms的验证者在游戏中拥有更多皮肤，使他们对自己的行为更负责任。
- **Amount of delegated Atoms:**要委托给验证者的Atoms的总额。一个高的投票权表明社区新任这个验证者，但是也意味着验证者对于黑客来说也是更大的目标。更大的验证器也减少了网络的分散化
- **Commission rate:** 在佣金被分发给委托人之前，佣金是委托人的收入。
- **Track record:** 跟踪记录。委托人可能会看验证者计划委托的跟踪记录。这包括排行、关于提案的过去投票记录，历史平均正常运行时间，节点被破坏的频率。

除了这些标准外，验证者有可能表明一个网站地址去完成他们的简介。验证者需要以某种方式建立声誉以吸引委托人。例如，对于验证者来说让第三方机构审核他们的组织是一个很好的实践。请注意，Tendermint团队不会对他们批准或进行任何审计。

## Responsibilities

-----

### Do validators need to be publicly identified?

验证者需要公开确定吗？

不需要，每个委托人都会根据自己的标准评估验证人。验证人在提名时可以注册网站地址，以便他们可以在他们认为合适的情况下宣传他们的操作。一些委托人可能更喜欢一个网站，清楚地显示运行验证者的团队及其简历，而其他人可能更喜欢具有良好记录的匿名验证人。

### What are the responsibilities of a validator?

验证者有两个主要的职责:

- 能够不断运行正确版本的软件:验证者需要确保他们的服务器总是在线的，并且他们的私钥没有被损坏。
- 积极参与治理：验证者被要求对每一个提案投票。

另外，验证者被期待成为社区中活跃的成员。他们应始终与生态系统的当前状态保持同步，以便他们可以适应任何变化。

### What does 'participate in governance' entail?

参与治理意味着什么？

在Cosmos Hub中的验证者和委托者可以对提案进行投票来改变操作参数（例如区块gas限制），协调更新，或者对于任何给与的问题做一个决策。

验证者在治理系统中扮演着一个特殊的角色。是系统的支柱，他们被要求对每一个系统进行投票。这一点尤其重要，因为不投票的委托人将继承其验证人的投票权。

### Can a validator run away with their delegators' Atoms?

验证者能否拿着委托人的Atoms一起逃跑？

通过委托一个验证者，一个用户委托投票权。一个验证者的投票权越多，他们在共识和治理过程的权重越大。并不意味着验证者保管他们委托人的Atoms。也不意味着一个验证者可以拿走委托人的资金逃跑。

即使委托的资金也不能被他们的验证者偷走，如果他们的验证者行为不端，委托人也是有责任的。

### How often will a validator be chosen to propose the next block? Does it go up with the quantity of bonded Atoms?

一个验证者选择多久提出下一个区块？是否与绑定的Atoms的数量有关？

被选择提出下一个区块的验证者叫做提议者。每个提议者被确定地选择，被选择的频率是和投票量（例如，绑定的Atoms的数量）是成比例的。例如，如果所有验证人的总绑定股份是100Atoms，并且每个验证者的总股份是10Atoms,然后这个验证者将提出约10％的区块。

### Will validators of the Cosmos Hub ever be required to validate other zones in the Cosmos ecosystem?

Cosmos Hub的验证人员是否需要验证Cosmos生态系统中的其他空间？

他们会的。如果治理决定这样，Cosmos Hub的验证者可能要求去验证Cosmos生态系统的其他空间。

## Incentives

----

### What is the incentive to stake?-什么是质押激励？

一个验证者资金池的每个成员获得不同类型的收入：

- **Block rewards:**（区块奖励），由验证者运行的应用程序的原生token（例如Cosmos Hub上的Atoms）被膨胀以产生区块供给。这些供给存在去激励Atom持有者绑定他们的资产，因为没有绑定的Atom随着时间的流逝将会被稀释。
- **Transaction fees:** (交易费)Cosmos Hub包含一个被接受作为费用支付的币的白名单列表。最初的费用币是`atom`.

验证者资金池中的所有收入根据每个验证人的的权重被分配。然后，每个验证者资金池中的收入按照委托资金的比例来分配。委托人收入佣金会在分发之前由验证人应用。 

### What is the incentive to run a validator ?-成为一个验证者的动机是什么？

由于佣金，验证人的收入比其委托人的收入要多。

验证者在治理方面也扮演着一个主要角色。如果一个委托者不投票，他们从验证者那里继承投票。这在生态系统中给验证者很大的责任。

### What are validators commission?-什么是验证者佣金？

验证者资金池的收入被验证者和他们的委托者分配。验证者可以对要给委托人的那部分收入应用一个佣金比率。这个佣金被设置作为一个百分比。每个验证者自由地设置他们最初的佣金率，最大的佣金日变化率，最大的佣金率，Cosmos Hub强制每个验证者设置参数。只有验证者被创建之后，佣金率才可以改变。

### How are block rewards distributed?-区块奖励如何分发？

区块奖励根据他们的投票权按比例分发给所有的验证者。这就意味着即使每个验证者获得每个奖励的atom，所有的验证者最终将会保持相等的权重。

让我们举个例子，有10个验证者拥有相同的投票权，和一个1%的佣金率。我们假定一个区块的奖励是1000 Atom，每个验证者有20%的自我绑定Atoms。这些tokens不直接提交给提议者。相反，它们在验证者中均匀分布。因此，现在每个验证者资金池中都有100个Atoms。这些100 个Atoms将会根据每个参与者资金的比例来分配：

- Commission: `100*80%*1% = 0.8 Atoms`
- Validator gets: `100\*20% + Commission = 20.8 Atoms`
- All delegators get: `100\*80% - Commission = 79.2 Atoms`

然后，每个委托人按照他们质押在验证者资金池的比例获得79.2 Atoms的一部分。

### How are fees distributed?-交易费如何分配？

费用类似地分配，除了区块提议者可以获得他们提议的区块的费用奖励，如果它们包括超过严格的最低要求的预先提交。

当一个验证者被选中区提交下一个区块时，他们必须包括上一个区块至少2/3的预提交。然而，`激励以奖金的形式包括超过2/3的预先提交`。这个奖金是线性的：范围从1％（如果提议者包括2/3预先提交）（块有效的最小值）到5％（如果提议者包含100％预先提交）。当然，一个提议者不应该等太久或者其他验证者可能超时，然后转到下一个提议者。因此，验证者必须在等待时间去得到大部分的签名和失去提出下一个区块的风险之间找到一个平衡。这个机制的目标是激励非空的区块提案，验证者之间更好的联网以及减轻审查。

让我们举一个具体的例子去说明前面提到的概念。在这个例子中，有10个验证者拥有相同的资金。他们中的每个人都应用1%的佣金率，并且有20%的自我委托的Atoms。现在，出现一个成功的区块有1025.51020408 Atoms 费用。

首先，一个2%的 `tax`被应用。相应的Atoms放入储备池中。储备池中的资金通过治理被分配用来资助悬赏和更新。

- `2% * 1025.51020408 = 20.51020408` 这部分放到储备资金池里。

现在还剩余1005个Atoms。我们假定提议者100%对这个区块进行了签名。那么他将获得奖励的5%（这5%是每个验证者获得奖励的5%，作为提议者他会多拿奖励的5%）

We have to solve this simple equation to find the reward R for each validator:

计算每个验证者的奖励R。

`9*R + R + R*5% = 1005 ⇔ R = 1005/10.05 = 100`

- For the proposer validator:（对于提议验证者）
  - The pool obtains `R + R * 5%`: 105 Atoms（多获得奖励的5%）
  - Commission: `105 * 80% * 1%` = 0.84 Atoms（验证者从委托人那里拿到的佣金）
  - Validator's reward: `105 * 20% + Commission` = 21.84 Atoms（验证者奖励要加上佣金）
  - Delegators' rewards: `105 * 80% - Commission` = 83.16 Atoms (each delegator will be able to claim its portion of these rewards in proportion to their stake)
  - 委托人奖励：要减去佣金，（每个委托人根据他们委托的比例分配这部分委托人奖励)
- For each non-proposer validator:(对于非提议验证者)
  - The pool obtains R: 100 Atoms
  - Commission: `100 * 80% * 1%` = 0.8 Atoms
  - Validator's reward: `100 * 20% + Commission` = 20.8 Atoms
  - Delegators' rewards: `100 * 80% - Commission` = 79.2 Atoms (each delegator will be able to claim their portion of these rewards in proportion to their stake)

### What are the slashing conditions?-什么是惩罚情况？

如果一个验证者行为不端，他们委托的资金将会部分地削减。目前有两个错误可能导致验证者及其委托人削减资金：

- **Double signing:**双重签名。如果有人报告在链A上，一个验证者在同一高度在链A和链B上签名了两个区块，并且如果链A和链B共享一个相同的祖先，那么在链A上的验证者将会得到5%的惩罚。
- **Downtime:** 停机。如果一个验证者错过了超过最后10个区块的95%，那么他将会获得0.01%的惩罚。

### Do validators need to self-delegate Atoms?

委托人需要自我委托 Atoms吗？

他们至少需要自我委托`1 atom`。即使没有义务像验证者一样自我委托超过`1 atom`，委托人应该想让他们的验证者有更多的自我委托资金在他们的资金池中。换句话说，验证者在游戏中应该有皮肤。

为了让委托人对他们的验证者在游戏中有多少皮肤有保障，后者可以示意一个最小的自我委托的 Atoms。如果一个验证者的自我委托资金低于预定义的限制，这个验证者和他的所有委托人将会解绑。

### How to prevent concentration of stake in the hands of a few top validators?

如何防止几个顶级验证人手中的股权集中？

目前，社区应该以聪明和自我保护的方式运行。当一个比特币矿池获得了太多的挖矿权，社区通常会停止对这个矿池的贡献。Cosmos Hub最初将依赖于相同的效果。还有其他机制可以尽可能地平滑这个过程：

- **Penalty-free re-delegation:**(无罚款 再委托)，这就允许委托人很容易地从一个验证者转换到另一个，目的是为了减少验证者粘性。
- **UI warning:** (UI 警告)，用户如果想去委托资金给一个拥有重大质押权的验证者，他们将会被 Cosmos Voyager警告。

## Technical Requirements-技术要求

-----

### What are hardware requirements?-什么是硬件要求？

验证者应该期望为一个或多个数据中心位置配置冗余电源，网络，防火墙，HSM和服务器。

我们期望适度的硬件规格在开始时将会是需要的，并且他们随着网络使用的增加可能上升。加入测试网是了解更多的最好方式。

### What are software requirements?-什么是软件需求？

为了运行一个 Cosmos Hub节点，验证者应该开发监视器，警报和管理解决法案。

### What are bandwidth requirements?-什么是带宽需求？

相比于以太坊和比特币这样的链， Csomos网络有更高的吞吐量。

我们推荐数据中心节点仅仅连接在云端可信的全节点或者其他在社交上相互了解的验证者。这减轻了数据中心节点拒绝服务攻击的负担。

最终，随着网络使用量的增加，每天多GB的带宽非常实际。

### What does running a validator imply in terms of logistics?

成功的验证者操作将需要多个高技能人员的努力和持续的操作注意力。例如，比运行比特币矿工要多得多。

### How to handle key management?-如何处理秘钥管理？

验证者应该期待运行一个支持 ed25519秘钥的 HSM（哈希算法）。这有潜在的操作：

- YubiHSM 2
- Ledger Nano S
- Ledger BOLOS SGX enclave
- Thales nShield support

Tendermint团队不推荐上面的解决方案。鼓励社区加强改进HSM和密钥管理安全的努力。 

### What can validators expect in terms of operations?-验证者在操作方面可期待什么？

运行一个有效的操作是避免不期待的解绑或者被削减的关键。这包括能够响应攻击，中断，以及维护数据中心的安全性和隔离性。

### How can validators protect themselves from denial-of-service attacks?

验证者如何保护自己免受拒绝服务攻击？

当攻击者向IP地址发送大量互联网流量以防止IP地址的服务器连接到互联网时，就会发生拒绝服务攻击。

一个攻击者扫描网络，试图去了解各个验证者节点的IP地址，通过发送大量互联网流量断开他们的连接。

减轻这些风险的一种推荐方法是验证者在所谓的岗哨节点架构中仔细构建其网络拓扑。

验证者节点应该连接到他们信任的全节点，因为他们操作他们自己或者运行在社交上其他熟悉的验证者节点。一个验证者节点一般运行在一个数据中心。大多数数据中心提供与主要云提供商网络的直接链接。验证者可以使用这些链接去连接到云端的哨兵节点。这直接把拒绝服务的负担从验证人节点转移到他的哨兵节点，并且可能需要转换或激活新的哨兵节点以减轻对现有节点的攻击。

哨兵节点可以快速的转换或者改变他们的IP地址。因为指向哨兵节点的链接位于私有IP空间中，一个以网络为依赖的攻击不能直接干扰他们。这将会确保验证者区块提案和投票总是带到网络的其他地方。

预计验证人那部分的良好操作程序将完全缓解这些威胁。

For more on sentry node architecture, see [this](https://forum.cosmos.network/t/sentry-node-architecture-overview/454).





