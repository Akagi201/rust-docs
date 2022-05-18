<!-- markdownlint-disable MD010 -->
# Polkadot 协议信息

> 原文链接：<https://wiki.polkadot.network/docs/build-protocol-info>
>
> 翻译：[Akagi201](https://github.com/Akagi201)

本页是对 Polkadot 协议的高层次介绍，包括 Polkadot 特有的术语，与你可能使用过的其他链的显著区别，以及与链交互的实用信息。

## 代币

* Token decimals:
  * Polkadot(DOT): 10
  * Kusama(KSM): 12
* Base unit: "Planck"
* Balance 类型：[`u128`](https://doc.rust-lang.org/std/u128/index.html)

### 重定货币单位 (Redenomination)

Polkadot 进行了一次投票，该投票于 2020 年 7 月 27 日结束（区块 888_888），其中利益相关者决定对 DOT 代币进行重新命名。重定货币单位并不改变网络中的基本单位（在 Polkadot 中称为 "plancks"）的数量。唯一的变化是，单个 DOT 代币将是 1e10 plancks，而不是原来的 1e12 plancks。请看 Polkadot 博客文章，[解释细节](https://medium.com/polkadot-network/the-first-polkadot-vote-1fc1b8bd357b)和[投票结果](https://medium.com/polkadot-network/the-results-are-in-8f6b1ca2a4e6)。

重定货币单位在转账启用 72 小时后生效，在 1_248_326 区块，大约发生在 2020 年 8 月 21 日 16:50 UTC。

## 地址

在 Polkadot（和大多数 Substrate 链）中，用户账户由 32 字节（256 位）的 `AccountId` 来识别。这通常是，但不总是，一个加密密钥对的公钥。

Polkadot（和 Substrate）使用 SS58 地址格式。这是一个广泛的 "meta-format"，旨在处理许多不同的加密方案和链。它与比特币的 Base58Check 格式有很多共同之处，如版本前缀，基于哈希的校验后缀，以及 base-58 编码。

有关编码信息和更全面的网络前缀列表，请参见 Substrate 文档中的 [SS58 页面](https://docs.substrate.io/v3/advanced/ss58/)。

> ### 不要使用正则来验证地址
>
> 始终使用地址的前缀和校验和来验证。Substrate API Sidecar 提供了一个 `accounts/{accountId}/validate` 路径，为提供的地址返回一个布尔值 `isValid` 响应。如果你想用其他方式验证地址，请看[验证文档](https://docs.substrate.io/v3/advanced/ss58/#validating-addresses)。

与本文档相关的 SS58 前缀

* Polkadot: 0
* Kusama: 2
* Westend: 42

### 密码学

Polkadot 支持以下[加密密钥对](https://wiki.polkadot.network/docs/learn-cryptography)和签名算法：

* Ed25519
* Sr25519 - 在 Ristretto 组的 Schnorr 签名
* 在 secp256k1 上的 ECDSA 签名

请注意，secp256k1 密钥的地址是公钥哈希值的 SS58 编码，以便将公钥从 33 字节减少到 32 字节。

## 存在性存款 (Existential Deposit)

Polkadot，以及大多数基于 Substrate 的链，使用存在性存款（ED）来防止灰尘账户 (dust account) 膨胀链的状态。如果一个账户下降到 ED 以下，它将被收割，即从存储中完全删除，并且重置 nonce。Polkadot 的 ED 是 1 DOT，而 Kusama 的是 33.3333 microKSM（0.00003333 KSM）。你总是可以通过检查常数`balances.existentialDeposit` 的[链状态](https://polkadot.js.org/apps/#/chainstate)来验证存在性存款。

同样，如果你发送一个价值低于 ED 的转账到一个新的账户，将会失败。托管钱包应该设置一个大于 ED 的最低提款金额，以保证成功提款。

出于审计目的而跟踪账户 nonces 的钱包和托管人应该注意不要让账户被收割，因为用户可以退还地址并尝试用它进行交易。Balances pallet 提供了一个 `transfer_keep_alive` 函数，如果这样做会导致收割发送者的账户，该函数将返回一个错误并中止，而不是进行转账。

> ### 存在性存款是一个中继链特性
>
> 你在中继链上的账户对 parachains 没有直接影响，因为你在每个 parachain 上都有独立的账户。不过，parachains 能够定义他们自己的存在性存款，但这与中继链的 ED 是分开的。

> ### STATEMINT 的存在性存款
>
> Statemint parachain 的存在性存款（0.1 DOT）比 Relay Chain（1 DOT）低，同时交易费用也较低。强烈建议在 Statemint 上处理余额转账。Statemint 的整合将在本指南的下一页讨论。

## Free vs. Reserved vs. Locked vs. Vesting Balance

账户余额信息存储在 AccountData 中。Polkadot 主要处理两种类型的余额：`free` 和 `reserved`。

对于大多数操作来说，`free` 余额是你所感兴趣的。例如，它是一个账户在质押和治理方面的 "权力"。`reserved`余额代表被某些操作预留的资金，仍然属于账户持有人，但不能使用。

`lock` 是对 `free` 余额的一种抽象，它可以防止为某些目的而花费。几个锁可以在同一个账户上操作，但它们是重叠的而不是相加的。当在网络上完成任务时，锁会自动添加到账户上（例如，租赁一个 parachain 插槽或投票），这些是不可定制的。例如，一个账户可以有 200DOT 的`free`余额，上面有两个锁。150DOT 用于`Transfer`，100DOT 用于`Reserve`。该账户不能进行转账，使其 `free` 余额低于 150DOT，但一个操作可能导致 `reserve` DOT，使`free`余额低于 150，但高于 100DOT。

用于抵押的绑定代币和治理公投中的投票都利用了锁。

`Vesting` 是另一个使用`free`余额锁的抽象概念。`Vesting`设置了一个锁，随着时间的推移不断减少，直到所有的资金都可以转移。

更多信息：

* [LockableCurrency](https://docs.substrate.io/rustdocs/latest/frame_support/traits/trait.LockableCurrency.html)
* [WithdrawReasons](https://docs.substrate.io/rustdocs//latest/frame_support/traits/struct.WithdrawReasons.html)
* [Vesting](https://docs.substrate.io/rustdocs/latest/pallet_vesting/index.html)

## 外部交易和事件

### 外部交易

外部交易构成来自外部世界并且呈现三种形式：

* 固有交易 (Inherents)
* 签名交易
* 无签名交易

作为一个基础设施供应商，你将几乎完全与签名交易打交道。然而，你会在你解码的区块中看到其他的外部交易。在 [Substrate 文档](https://docs.substrate.io/v3/concepts/extrinsics/)中找到更多信息。

固有交易 (Inherents) 包含的信息不能被证明是真实的，但验证者基于某种合理性的措施而同意。例如，一个时间戳不能被证明，但验证者可以同意它是在他们的系统时钟的某个误差内。固有交易 (Inherents) 不会在网络上 gossiped 传播，只有区块作者才会将其插入区块中。

签名交易包含了发出交易的账户的签名，并为将交易纳入链中而支付费用。因为在执行之前就可以认识到将签名交易纳入链上的价值，所以它们可以在网络上的节点之间进行 gossiped，垃圾信息的风险很低。签名的交易符合以太坊或比特币的交易概念。

有些交易不能由付费账户签署，而使用无签名交易。例如，当用户从以太坊 DOT 指标合约中认领他们的 DOT 到一个新的 DOT 地址时，新地址还没有任何资金可以用来支付费用。

### 交易死亡率

外部交易可以是会死的或不死的。交易的负载包括一个区块编号和区块 hash 检查点，从该检查点开始，交易是有效的，还有一个有效期（在某些地方也称为 "era"），代表检查点之后的区块数量，该交易是有效的。如果在这个有效期内的区块中没有包含外部交易，它将被从交易队列中丢弃。

链上只存储有限数量的先前块 hash 值作为参考。你可以从链状态或元数据中查询这个参数，称为 `BlockHashCount`。这个参数在创世时被设置为 2400 个区块（约 4 小时）。如果有效期大于链上存储的区块数量，那么只要有一个区块可以检查，即有效期和区块哈希数的最小值，交易就会有效。

将区块检查点设置为零，使用创世哈希值，有效期为零，将使交易 "不死的"。

注意：如果一个账户被收割，用户重新为该账户提供资金，那么他们可以重放一个不死的交易。总是默认使用会死的外部交易。

### 外部交易的唯一 ID

> #### 注意
>
> 假设交易的哈希值是一个唯一的标识符，是索引服务和保管人犯的头号错误。这个错误会给你的用户带来重大问题。请确保你仔细阅读本节。

现有区块链上的许多基础设施供应商，例如以太坊，将交易的哈希值视为唯一的标识符。在 Polkadot 等基于 Substrate 的链中，交易的哈希值只作为交易中信息的指纹，有时两个具有相同哈希值的交易都是有效的。在其中一个无效的情况下，网络会妥善处理该交易，不向发送者收取交易费用，也不考虑该交易在区块中的完整性。

想象一下这个伪造的例子，有一个[被收割的账户](https://wiki.polkadot.network/docs/build-protocol-info#existential-deposit)。第一笔和最后一笔交易是相同的，而且都是有效的。

| Index 	| Hash 	| Origin    	| Nonce 	| Call                	| Results                       	|
|-------	|------	|-----------	|-------	|---------------------	|-------------------------------	|
| 0     	| 0x01 	| Account A 	| 0     	| Transfer 5 DOT to B 	| Account A reaped              	|
| 1     	| 0x02 	| Account B 	| 4     	| Transfer 7 DOT to A 	| Account A created (nonce = 0) 	|
| 2     	| 0x01 	| Account A 	| 0     	| Transfer 5 DOT to B 	| Successful transaction        	|

此外，在基于 Substrate 的链中，并非每个外在交易都来自于作为公钥/私钥对的账户；相反，Substrate 有调度 "origin" 的概念，它可以由公钥账户创建，但也可以由治理等其他方式形成。这些 origin 并不像账户那样有一个与之相关的 nonce。例如，治理可能会多次调度具有相同参数的相同调用，如 "将验证器集增加 10%"。这个调度信息（以及因此它的哈希值）将是相同的，哈希值将是该调用的可靠代表，但它的执行将有不同的效果，这取决于调度时链的状态。

在基于 Substrate 的链上唯一识别一个外部交易的正确方法是使用块 ID（高度或哈希值）和外部交易的索引。Substrate 将一个区块定义为一个头和一个外部交易的数组；因此，数组中位于规范链高度的索引将始终唯一地识别一个交易。这种方法反映在 Substrate 代码库本身中，例如，从 Multisig pallet 中引用一个[先前的交易](https://docs.substrate.io/rustdocs/latest/pallet_multisig/struct.Timepoint.html)。

### 事件

外部交易代表来自外部世界的信息，而事件则代表来自链的信息。外部交易可以触发事件。例如，当索取质押奖励时，Staking pallet 会发出 `Reward` 事件，告诉用户账户被记入了多少钱。

如果你想监控存款到一个地址，请记住，几个交易可以启动余额转账（如 `balances.transferKeepAlive` 和 `utility.batch` 交易，里面有一个转账）。仅仅监控余额转账交易是不够的。确保你监测每个区块中包含你感兴趣的地址的事件。监控事件而不是交易名称，以确保你能正确地记入存款。

### 手续费

Polkadot 使用基于权重的手续费，与 gas 不同，它是在调度前收取的。用户还可以添加 "小费"，以增加拥挤时期的交易优先权。更多信息见[交易费](https://wiki.polkadot.network/docs/learn-transaction-fees)页面。

### 编码

Parity 的集成工具应该允许你处理解码后的数据。如果你想绕过它们，直接与链上的数据互动，或者实现你自己的编解码器，Polkadot 使用 [SCALE 编解码器](https://docs.substrate.io/v3/advanced/scale-codec/)对块和交易数据进行编码。

## 运行时升级

[运行时升级](https://wiki.polkadot.network/docs/learn-runtime-upgrades)允许 Polkadot 改变链的逻辑而不需要硬分叉。硬分叉将需要节点操作员手动将他们的节点升级到最新的运行时版本。在一个分布式系统中，这是一个复杂的协调和沟通过程。Polkadot 可以不通过硬分叉进行升级。现有的运行时逻辑被遵循，以将存储在区块链上的 Wasm 运行时更新为新版本。然后，该升级被包含在区块链本身中，这意味着网络上的所有节点都会执行它。

一般来说，在运行时升级之前，不需要手动升级你的节点，因为它们会自动开始遵循链的新逻辑。只有当运行时需要新的主机功能或者网络或共识发生变化时，才需要更新节点。

为一个给定的运行时版本构建的交易将不能在以后的版本上运行。因此，基于一个运行时版本构建的交易在以后的运行时版本中是无效的。如果你认为你不能在升级前提交一个交易，那么最好等待，在升级发生后再构建。

虽然升级你的节点一般不需要跟随升级，但我们建议跟随 Polkadot 的发布，及时升级，尤其是高优先级或关键的发布。

## 智能合约

波卡中继链不支持智能合约。

## 其他网络

除了运行一个私人网络，波卡还有另外两个网络，你可以在部署到 Polkadot 主网之前测试基础设施。

`Kusama Canary Network`: Kusama 是波卡的先行网络。许多冒险的特性会优先于 Polkadot 部署到 Kusama 上。

`Westend Testnet`: Westend 是波卡的测试网并且使用 Polkadot 运行时。

## 其他 FAQ

### 如果没有相应的、链上的交易，一个账户的余额可以改变吗？

不能，但不是所有的余额变化都在交易中，有些是在事件中。你将需要运行一个归档节点并监听事件和交易，以跟踪所有的账户活动。这尤其适用于锁定操作，如果你将余额计算为可花费的余额，即自由余额减去最大锁定。

### 什么链的深度被认为是 "安全的"？

Polkadot 使用一个确定性的最终性机制。一旦一个区块被最终确定，除非是硬分叉，否则它不能被逆转。Kusama 曾有过硬分叉，为了取消运行时的升级，不得不恢复四个最终化的区块。使用 10 个区块的最终性深度应该是安全的。

请注意，在 Polkadot 中，区块生产和最终化是孤立的过程，链上可以有一个很长的未最终化的头。

### 用户需要与任何智能合约互动吗？

不，用户直接与链上的逻辑互动。

### Polkadot 有状态租金吗？

没有，Polkadot 使用存在性存款来防止灰尘账户和其他经济机制，如锁定或保留代币的操作，利用状态。

### 什么是查看当前链高度的外部来源？

* [Polkadot-JS explorer](https://polkadot.js.org/apps/#/explorer)
* [Polkascan block explorer](https://explorer.polkascan.io/)
