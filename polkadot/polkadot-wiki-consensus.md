# Polkadot 共识

> 原文链接：<https://wiki.polkadot.network/docs/learn-consensus>
>
> 翻译：[Akagi201](https://github.com/Akagi201)

## 为什么我们需要共识？

共识是一种对共享状态达成协议的方法。为了使区块链的状态继续构建并向前发展，网络中的所有节点必须同意并达成共识。它是去中心化网络中的节点能够相互保持同步的方式。如果没有区块链中节点的去中心化网络的共识，就没有办法确保一个节点认为是真实的状态会被其他节点所共享。共识的目的是在对网络有各自主观看法的参与者中提供客观的状态观点。它是这些节点进行沟通并达成协议的过程，并能够构建新的区块。

## 什么是 PoW 和 PoS？

工作证明（PoW）和权益证明（PoS）被不准确地用作区块链共识机制的简称，但这并没有抓住全貌。PoW 是商定区块作者的方法，也是更全面的中本聪共识的一部分，其中还包括链选择算法（比特币中最长的链规则）。同样，PoS 是一套选择验证人的规则，并没有指定一个链的选择规则或一个链如何达到最终结果。PoS 算法传统上是与节点之间达成拜占庭协议的算法相搭配的。例如，Tendermint 是一种实用的拜占庭容错算法，它使用 PoS 作为其验证人集选择方法。

## 为什么不用工作量证明？

虽然在达成关于下一个区块生产者的分散共识方面简单而有效，但中本聪共识的工作证明消耗了大量的能源，没有经济或可证明的最终结果，也没有抵制卡特尔的有效策略。

## 委托权益证明

在传统的 PoS 系统中，区块生产的参与取决于代币持有量，而不是计算能力。虽然 PoS 开发者通常有一个主张以去中心化的方式公平参与，但大多数项目最终都提出了某种程度的中心化操作，其中拥有完全参与权的验证人的数量是有限的。这些验证人常被认为是最富有的，因此，影响 PoS 网络，因为他们的赌注最大。通常，拥有必要知识（和设备）来维护网络的候选人数量有限；这也会直接增加运营成本。拥有大量验证人的系统倾向于形成矿池，以减少其收入的差异，并从规模经济中获利。这些矿池往往是链外的。

缓解这种情况的方法是在链上实现矿池的形成，并允许代币持有者 [用他们的权益] 对代表他们的验证人进行投票。

Polkadot 使用 NPoS（Nominated Proof-of-Stake）作为其选择验证人组的机制。它被设计成验证人和提名人的角色，以最大限度地提高链的安全性。对维护网络感兴趣的行为者可以运行一个验证者节点。

验证人承担着在 BABE 中生产新区块、验证准区块和保证最终性的角色。提名人可以选择用他们的权益来支持选定的验证人。提名人可以批准他们信任的候选人，并用他们的代币支持他们。

## 概率性 vs 可证明的最终一致性

一个运行 PoW 的纯中本共识区块链只能实现概率最终性的概念并达成最终共识。概率最终性是指在关于网络和参与者的一些假设下，如果我们看到几个区块建立在一个给定的区块上，我们可以估计它是最终的概率。最终共识是指在未来的某个时间点，所有节点都会对一组数据的真实性达成共识。这种最终的共识可能需要很长的时间，并且将无法提前确定它需要多长时间。然而，最终一致性程序，如 GRANDPA（基于 GHOST 的递归 ANcestor 衍生前缀协议）或以太坊的 Casper FFG（友好的最终一致性程序），旨在对区块的最终性给予更强和更快的保证 -- 具体而言，在发生了一些拜占庭协议的过程后，它们永远无法被逆转。不可逆转的共识的概念被称为可证明的最终性。

在 [GRANDPA 论文](https://github.com/w3f/consensus/blob/master/pdf/grandpa.pdf)中，它是这样表述的：

> ### 注意
> 
> 我们说在协议中的一个 oracle A 是最终一致性，当他在某个指定时间后对所有参与者返回相同的值。

## 混合共识

当我们谈论 Polkadot 的共识协议时，有两个协议，GRANDPA 和 BABE（Blind Assignment for Blockchain Extension）。我们谈论这两种协议是因为 Polkadot 使用的是所谓的混合共识。混合共识将最终一致性程序从区块生产机制中分割出来。

这是一种在 Polkadot 中获得概率最终一致性（总是能够产生新的区块）和可证明最终一致性（在经典链上有一个普遍的协议，没有机会逆转）的好处的方法。它还避免了每种机制的相应缺点（在概率最终一致性中有可能不自觉地跟随错误的分叉，在可证明最终一致性中有可能 "停滞"--无法产生新区块）。通过结合这两种机制，Polkadot 允许快速生产区块，而较慢的最终一致性机制则在一个单独的进程中运行，以最终确定区块，而不会有较慢的交易处理或停顿的风险。

混合共识在过去已经被提出。值得注意的是，在 [EIP 1011](https://eips.ethereum.org/EIPS/eip-1011) 中，它被作为以太坊过渡到股权证明的一个步骤提出（现已失效），其中指定了 [Casper FFG](https://wiki.polkadot.network/docs/learn-consensus#casper-ffg)。

## 区块生成: BABE

BABE（Blind Assignment for Blockchain Extension）是在验证者节点之间运行的区块生成机制，决定了新区块的作者。BABE 作为一种算法可与 [Ouroboros Praos](https://eprint.iacr.org/2017/573.pdf) 相媲美，但在链的选择规则和槽的时间调整方面有一些关键的区别。BABE 根据股权并使用 Polkadot 随机性循环将区块生产槽分配给验证者。

Polkadot 中的验证者将参与每个槽的抽签，这将告诉他们是否是该时段的区块生产者候选人。槽是不连续的时间单位，名义上是 6 秒的长度。由于这种随机性机制，多个验证者可能是同一槽位的候选人。其他时候，一个槽可能是空的，导致区块时间不一致。

### 一个槽位多个验证人

当多个验证者在一个给定的时段内成为区块生产者候选人时，所有验证者都会生产一个区块并将其广播到网络上。在这一点上，这是一场竞赛。谁的区块首先到达网络的大部分地区，谁就获胜。根据网络的拓扑结构和延迟，两个链将继续以某种方式建立，直到最终一致性程序启动并切断一个分叉。请看下面的 "分叉选择"，了解它的工作原理。

### 一个槽位上没有验证人

当没有验证人在随机性抽签中掷出足够低的分数以获得生产区块的资格时，一个槽位可能仍然看起来没有区块。我们通过在后台运行一个次要的、循环式的验证人选择算法来避免这种情况。通过这种算法选择的验证人总是产生区块，但是如果同一个槽位也从 [VRF 选择的](https://wiki.polkadot.network/docs/learn-randomness#vrf)验证器中产生一个主要区块，那么这些次级区块就会被忽略。因此，一个槽位可以有一个主要的或次要的块，而且没有槽位被跳过。

关于 BABE 的更多细节，请参见 [BABE 论文](https://research.web3.foundation/en/latest/polkadot/block-production/Babe.html)。

### BADASS BABE: SASSAFRAS

SASSAFRAS（Semi Anonymous Sortition of Staked Assignees For Fixed-time Rhythmic Assignment of Slots）（又称 SASSY BABE 或 BADASS BABE），是 BABE 的一个扩展，作为一个恒定时间块生产协议。这种方法试图解决 BABE 的缺点，确保以时间恒定的间隔精确地生产一个块。该协议利用 zk-SNARKs 来构建一个环形 VRF，是一项正在进行的工作。本节将随着进展而更新。

## 最终一致性程序: GRANDPA

GRANDPA（GHOST-based Recursive ANcestor Deriving Prefix Agreement）是为 Polkadot 中继链实现的最终一致性程序。

只要有 2/3 的节点是诚实的，它就能在部分同步的网络模型中工作，并能在异步的环境中应对 1/5 的拜占庭节点。

一个值得注意的区别是，GRANDPA 在链上而不是在块上达成协议，大大加快了最终一致性确定的过程，即使在长期的网络分区或其他网络故障之后。

换句话说，只要有超过 2/3 的验证者证明某条链包含某个区块，那么导致该区块的所有区块就会被一次性敲定。

### 协议

请参考 [GRANDPA 论文](https://github.com/w3f/consensus/blob/master/pdf/grandpa.pdf)，了解协议的完整描述。

### 实现

[Substrate GRANDPA 实现](https://github.com/paritytech/substrate/tree/master/frame/grandpa)是 Substrate FRAME 的一部分。

## 分叉选择

将 BABE 和 GRANDPA 结合起来，Polkadot 的分叉选择就变得很清楚了。BABE 必须总是建立在已经被 GRANDPA 最终确定的链上。当在最终确定的头之后有分叉时，BABE 通过建立在拥有最多主要区块的链上提供概率上的最终性。

![consensus_fork_choice](assets/consensus_fork_choice.png)

在上图中，黑色区块是最终确定的，而黄色区块不是。标有 "1" 的区块是一级区块；标有 "2" 的区块是二级区块。尽管最上面的链是最新完成的区块上最长的链，但它并不符合条件，因为在评估时它的主块比下面的主块少。

## 对比

### 中本聪共识 (Nakamoto)

中本聪的共识由最长链规则组成，使用工作量证明作为其抵抗女巫攻击机制和领导人选举。

中本聪共识只给了我们概率上的最终一致性。概率最终一致性指出，过去的一个区块的安全程度仅与它的确认数量有关，或在它上面建立的区块数量。当更多的区块被建立在工作量证明链中的特定区块之上时，这个特定的链背后已经花费了更多的计算工作。然而，这并不能保证包含该区块的链将始终是商定的链，因为一个拥有无限资源的行为者有可能建立一个竞争链，并花费足够的计算资源来创建一个不包含特定区块的链。在这种情况下，比特币和其他工作量证明链采用的最长链规则将转移到这个新的链上来作为经典的链。

### PBFT / Tendermint

请参考 [Cosmos 对比文章](https://wiki.polkadot.network/docs/learn-comparisons-cosmos#consensus)中相关内容

### Casper FFG

GRANDPA 和 Casper FFG 之间的两个主要区别是：

* 在 GRANDPA 中，不同的投票者可以同时对不同高度的区块进行投票
* GRANDPA 只依赖于最终确定的区块来影响底层区块生产机制的分叉选择规则

## 资源

* [BABE 论文](https://research.web3.foundation/en/latest/polkadot/block-production/Babe.html) - BABE 协议的学术描述。
* [GRANDPA 论文](https://github.com/w3f/consensus/blob/master/pdf/grandpa.pdf) - GRANDPA 最终一致性程序的学术描述。
* [Rust 实现](https://github.com/paritytech/finality-grandpa) - 参考实现和对应的 [Substrate pallet](https://github.com/paritytech/substrate/tree/master/frame/grandpa)
* [区块生成和波卡中最终一致性](https://www.crowdcast.io/e/polkadot-block-production) - 解释 BABE 和 GRANDPA 如何共同制作和敲定 Kusama 的区块，by Bill Laboon。
* [波卡中区块生成和最终一致性](https://www.youtube.com/watch?v=1CuTSluL7v4&t=4s) - Bill Laboon 在 MIT Cryptoeconomic Systems 2020 的学术演讲，深入描述了 Polkadot 的混合共识模型。
