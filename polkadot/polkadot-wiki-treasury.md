# Polkadot 国库

> 原文链接：<https://wiki.polkadot.network/docs/learn-treasury>
>
> 翻译：[Akagi201](https://github.com/Akagi201)

金库是通过部分区块生成奖励、交易费、惩罚、[质押收益](https://wiki.polkadot.network/docs/learn-staking#inflation)等方式收集的资金池。

国库中的资金可以通过提出支出提案来使用，如果得到[议会](https://wiki.polkadot.network/docs/learn-governance#council)的批准，在分配之前将进入一个等待期。这个等待期被称为花费期，其持续时间取决于[治理](https://wiki.polkadot.network/docs/learn-governance)，目前默认为 24 天。国库试图在不耗尽资金的情况下，尽可能多地花费队列中的提案。

国库支付是一个自动过程。

* 如果国库的资金用完了，但仍有已批准的提案需要资助，这些提案就会被保留在已批准的队列中，并在下一个支出期获得资助。
如果国库在一个支出期结束时没有花费所有的资金，它就会遭受一定比例的资金烧毁 -- 从而造成通货紧缩的压力。这鼓励了 Polkadot 的治理系统花费国库中的资金。这个百分比目前在 Polkadot 上是 1%。
* 当利益相关者希望从国库中提出支出时，他们必须保留至少 5% 的拟议支出的押金（见下文的变化）。如果提案被拒绝，该保证金将被削减，如果被接受，则将被退回。

提案可能包括（但不限于）:

* 基础设施部署和持续运行。
* 网络安全运营（监测服务，持续审计）。
* 生态系统供应（与友好的链合作）。
* 营销活动（广告、付费功能、合作）。
* 社区活动和推广（聚会、比萨饼派对、黑客空间）。
* 软件开发（钱包和钱包集成，客户端和客户端升级）。

[议会](https://wiki.polkadot.network/docs/learn-governance#council)管理国库，资金如何使用取决于他们的判断。
