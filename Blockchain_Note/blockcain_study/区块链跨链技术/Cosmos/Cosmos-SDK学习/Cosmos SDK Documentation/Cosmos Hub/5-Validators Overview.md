# Validators Overview

## Introduction

------------

Cosmos Hub是以Tendermint为基础，依赖于在区块链上有责任提交新区快的验证者集合。这些验证者通过广播包含每个验证者的私钥签名的加密签名的投票，以此来加入共识协议。

验证者候选人可以绑定他们自己的Atom，并且让他们的Atoms委托或质押给那些token持有者。Cosmos Hub将会有100个验证者，但是随着时间的推移，根据预定的进度，将会增加到300个验证者，这些验证者通过他们被委托多少股份来确定，拥有最多股份的前100个候选人将会成为 Cosmos validators。

当区块准备好时，验证者和他们的委托人都可以获得Atom，并且通过执行 Tendermint 共识协议，token作为交易费。原来（Initially），交易费用Atoms支付，但是在将来，Cosmos生态系统中的任何token如果被治理列入白名单，将作为费用招标有效。注意，验证者可以对他们的委托者接收到的费用设置佣金（commission）作为额外的激励（incentive）。

如果验证者双重签名，或者经常离线，或者不参加治理，他们质押的Atoms(包括用户委托给他们的Atoms)将会被扣除。惩罚（penalty）依赖于他们违反(violation)的严重性(severity)。

## Hardware

------------

。。。。

。。。。

## Set Up a Website

------------

设置一个专用的验证者网站，在我们的论坛上表明想要成为验证者的意向。这是重要的，因为委托人

想知道他们把Atoms委托给实体的信息。