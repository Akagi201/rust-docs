# Polkadot 平行链开发指南

> 原文链接：<https://wiki.polkadot.network/docs/build-pdk>
>
> 翻译：[Akagi201](https://github.com/Akagi201)

## 开发平行链的首选概述

本指南将涵盖构建 parachain 或 parathread 的动机，可用于促进这一工作的工具，测试的步骤，以及最后，如何在 Polkadot 上启动你的网络。

## 为什么创建平行链？

Parachains 与中继链相连，并由中继链保障。它们受益于集合安全、深思熟虑的治理以及网络的异质分片方法的整体可扩展性。创建 parachain 可以被看作是创建一个第一层区块链，它有自己的逻辑，并在 Polkadot 生态系统中平行运行。

开发人员可以专注于创建最先进的链，利用 Polkadot 的下一代方法。parachain 可能是的一些例子是。

DeFi（去中心化金融）应用
数字钱包
IoT（物联网）应用
游戏
网络 3.0 基础设施
以及更多。

Polkadot 旨在成为对区块链极致主义的赌注，Polkadot 的异质多链方法的成功将在 Web 3.0 和去中心化系统的整体推进中起到关键作用。因此，Polkadot 的 parachain 模型在设计时相信，未来的互联网将有许多不同类型的区块链一起工作。