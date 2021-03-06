# XCM Part II: 版本和兼容性

> 原文链接：<https://polkadot.network/blog/xcm-part-two-versioning-and-compatibility/>
>
> 翻译：[Akagi201](https://github.com/Akagi201)

![xcm_cover_p2](assets/xcm_cover_p2.png)

在我写的[第一篇关于 XCM 的文章](https://polkadot.network/blog/xcm-the-cross-consensus-message-format/)中，我介绍了它的基本架构、目标以及如何将其用于一些简单的用例。在这里，我们将继续深入探索 XCM 的一个有趣的方面：XCM 如何随着时间的推移而改变，而不在它所要连接的网络之间造成破坏。

拥有一种共同的语言可以解决人类互动中的很多问题。它使我们能够一起工作，解决冲突，并记录信息供以后使用。但是，语言只有在它能够表达的概念中才是有用的，在一个不断变化的世界中，一种语言必须改变和调整它的概念范围，否则就有可能被废弃。

不幸的是，过于突然地改变一种语言会损害其主要目的 -- 促进人们之间的交流。既然语言必须改变，就必须有方法来管理这些改变，而不至于使新的形式被不熟悉的人所误解。在这方面，一个非常有用的发明是字典，它可以帮助记录和存档一种语言在某一时期的概念调色板，从而使后代能够更好地理解历史文本。一本词典的版本可以被认为是一种语言的正式 "版本"。

时代可能会改变，但这些问题仍然非常熟悉。正如我在上一篇文章中解释的那样，XCM 只不过是一种语言，尽管是一种非常专业的语言。它是共识系统相互交谈的一种手段，随着加密行业，特别是 Polkadot 生态系统对这种 XCM 的需求以惊人的速度发展，那么必须有一些手段来确保这些变化不会影响到 XCM 的最初目标：互操作性。我们现在需要解决的不仅仅是共识空间的互操作性，还有共识时间的互操作性。

## 🔮 版本

由于我们预计 XCM 的语言会随着时间的推移而改变，同时也在使用中，所以要采取的一个非常简单的预防措施是确保我们在实际的消息内容之前就确定我们正在通信的 XCM 的哪个版本。我们通过使用一些版本包装器类型来做到这一点，之所以这样命名是因为它们用版本包装了 XCM 消息或其中的一个组成部分。在 Rust 代码中，这看起来非常简单：

```rust
pub enum VersionedXcm {
    V0(v0::Xcm),
    V1(v1::Xcm),
    V2(v2::Xcm),
}
```

当 "over the wire"（或者说，在共识系统之间）发送时，XCM 总是被放在这个版本的容器中。这可以确保太老的系统能够安全地接收这些消息，并识别出消息的格式不被它们所支持。它还允许较新的系统识别并相应地解释较旧的消息。

不仅仅是 XCM 消息有版本，在 XCM 代码库中，我们还对 `MultiLocation`、`MultiAsset` 以及其相关类型进行了版本管理。这是因为当链的 XCM 逻辑被升级时，它们可能需要被存储并在以后被解释。如果没有版本控制，我们可能会试图将旧的 `MultiLocation` 解释为新的 `MultiLocation`，结果发现它是无法理解的（或者更糟糕的是，可以理解但与原来的意思不同）。

## 💬 兼容性和翻译

版本划分是第一步，它确保我们能够识别正在使用的语言版本。它并不确保我们能够解释它，当然也不确保它是我们优先使用的同一版本。这就是兼容性的问题所在。我们所说的 "兼容性 "是指能够继续解释和表达我们不喜欢的 XCM 的版本。

如果我们希望能够按照我们选择的时间表升级我们的网络及其 XCM 的版本，那么这种兼容性就变得相当重要，因为我们可能希望与其他尚未升级或确实已经升级的网络进行通信。这可以细分为向后兼容和向前兼容。基本上说，向后兼容性是指升级后的系统能够继续在传统的世界中运作，而向前兼容性是指传统系统能够继续在升级后的世界中运作的能力。

在我们的案例中，我们希望两者都有，但是有实际的限制：如果新版本的 XCM 提供了以前版本所没有的功能，期望旧系统能够解释这些信息是不现实的。这有点像试图将 "社交媒体"一词翻译成拉丁文，然后期望它能被凯撒大帝从表面上理解。有些概念根本无法在遗留的环境中表达。

同样，XCM 的重大变化可能会导致能力从其概念模型中删除。这种情况较少发生，但类似于将某些古老的术语翻译成现代的等价物的问题。有趣的是，["点"的古老含义](https://www.lexico.com/definition/dot#h70279322284300)可能是这里的一个例子（它曾经意味着一种相当特殊的金融捐赠形式）。

因此，新版本的 XCM 被设计成与旧版本和新版本基本兼容，但通常会有一些 XCM 信息在替代的上下文中根本没有意义，无法翻译。

## 🗣 实践交流

如前所述，我们确保所有独立存在的消息都包括一个版本标识符。这意味着在系统之间发送的消息或在存储中持续存在的消息。但它并不包括所有的消息、位置和资产 -- 作为其他数据的一部分而存在的数据不需要进行版本识别，因为它的版本可以从其上下文中推断出来。

版本识别和兼容性/翻译对于接收来自旧网络的信息或向新网络发送信息是有帮助的，但是 -- 单独来看 -- 在反过来的时候就不太有用。这是因为，从升级的网络接收信息的传统网络本身并没有能够将新的 XCM 翻译成它可以解释的形式的逻辑 -- 相反，这种逻辑只存在于发送方，它有翻译代码能够以传统术语重新表达新信息。

因此，发送网络必须负责确保其发送的信息能够被接收网络解释。具体来说，用于报文的 XCM 版本必须不比接收网络支持的 XCM 版本更新。

为此，Polkadot 和 Kusama 中继链、Statemint、Statemine、Shell 和任何其他基于 Substrate/Frame 及其 XCM 引擎的链，都保留了一个远程链所支持的 XCM 版本的注册表。每当这些链发送 XCM 消息时，它首先通过查询其注册表来决定以什么版本发送消息。它将消息翻译成发送方和接收方所支持的 XCM 版本中的较早版本。对于保持更新的链来说，大多数情况下，这些都是相同的、最新发布的版本，可提供 XCM 的全部功能集。

这个注册表通常是由治理程序决定和升级的，这有点麻烦和乏味，特别是随着潜在目的地的数量增加。出于这个原因，引入了版本跟踪。

## 🤝 版本协商

版本跟踪是 XCM 版本故事中的最后一块拼图。它的功能是消除任何追踪潜在目标链的 XCM 版本所需的链外或治理过程。相反，这个过程是在链上自主进行的。

从本质上讲，它的作用是允许一个网络使用 XCM 向另一个网络查询它所支持的 XCM 的最新版本，并在这一变化时得到通知。来自这一查询的答复允许有关网络填充和维护其版本注册表，确保信息以最新的可理解的版本发送。

具体来说，XCM 中有三个有价值的指令：`SubscribeVersion`，允许一个人要求另一个人通知他现在和将来的 XCM 版本；`UnsubscribeVersion`，取消该请求；`QueryResponse`，将一些信息从响应者网络返回到发起网络的一般手段。这就是它们在 Rust 中的样子：

```rust
enum Instruction {
    SubscribeVersion {
        query_id: QueryId,
        max_response_weight: u64,
    },
    UnsubscribeVersion,
    /* snip */
}
```

所以 `SubscribeVersion` 需要两个参数。第一个，`query_id` 是 `QueryId` 类型的，这只是一个整数，用来让我们识别和区分回来的响应。所有导致发送响应的 XCM 指令都有一个类似的手段，以确保它们的响应能够被识别并得到相应处理。第二个参数叫做 `max_response_weight`，是一个 `Weight` 值（也是一个整数），表示回复返回时我们应该花费的最大计算时间。和 `query_id` 一样，这将被放置在该指令产生的任何响应信息中，并且需要确保任何不可预测的、可变的权重成本至少可以在执行前被限制在一个最大值。没有这个，我们就无法得到回复信息可能需要的解释时间的上限，从而无法安排它的执行。

`UnsubscribeVersion` 作为一个指令是相当贫瘠的，主要是因为对于一个给定的位置，一次只允许有一个版本的订阅。这意味着取消的发生，除了起源寄存器的内容外，没有更多的东西可以识别它。

![xcm_version_negotiate](assets/xcm_version_negotiate.png)

版本注册表及其用法的说明。在这里，A 链（XCM 版本 2）与 E 链（XCM 版本 3）进行协商，最终发送了一个版本 2 的消息，E 在解释该消息之前会自动翻译成版本 3。

## 👂 回复

第三条需要注意的指令是 `QueryResponse`，这是一条非常通用的指令，允许一条链回复另一条链，并在这样做时报告一些信息。下面是它在 Rust 中的用法：

```rust
enum Instruction {
    QueryResponse {
        query_id: QueryId,
        response: Response,
        max_weight: u64,
    },
    /* snip */
}
```

我们已经知道三个参数中的两个，因为它们是由 `SubscribeVersion` 中提供的值填充的。第三个被称为`response`，包含我们关心的实际信息。它被放在一个新的 `Response` 类型中，它本身是一个网络可能希望用来通知另一个网络的几种类型的联合。在 Rust 中，它看起来是这样的：

```rust
pub enum Response {
    Null,
    Assets(MultiAssets),
    ExecutionResult(Result<(), (u32, XcmError)>),
    Version(XcmVersion),
}
```

就我们目前的目的而言，只需要 `Version` 这一项，尽管我们将在以后的文章中看到，其他项目在其他情况下是有用的。

## ⏱ 执行时间

一般来说，我们不要求 `QueryResponse` 指令用 `BuyExecution` 来购买自己的执行时间，因为（假设它们是有效的），首先是现在的解释网络要求它们被发送。同样地，我们认为 `SubscribeVersion` 大致上符合发送方和接收方的共同利益，所以不期望它需要被支付。在任何情况下，由于它所产生的响应的异步性和不可预测性，支付将相当难以计算。

## 🤖 自动化

虽然这些 XCM 指令允许网络完全使用链上逻辑来确定其对话者支持的最新版本，但仍然存在何时启动这种版本发现 "握手 "的问题。一般来说，在创建发送 XCM 的信道时不能这样做，因为运输信道的创建在概念上比 XCM 的创建要低，而 XCM 是可能通过该信道发送的一种（也许是许多）数据格式。在这里把水搅浑可能会损害分层设计的独立性。此外，一些交叉共识的传输协议根本不是基于通道的，这就排除了在其开始时进行版本协商的可能性。

在 Polkadot 中继链和 Statemint 等底层链中，解决方案是在消息需要打包发送但目的地的最新版本未知时自动启动这个版本发现过程。这有一个轻微的缺点，即第一批消息将在次优的 XCM 版本下发送，这将发生在收到版本响应之前。如果这是一个实际问题，那么治理部门可以介入，强制该目的地的 XCM 初始版本与默认版本不同（一般设置为生产中仍可预期的最早的 XCM 版本）。

## ⌨️ XCM 内部的代码兼容性

关于版本问题的最后一点是代码编写。与 XCM 的 `over-the-wire` 格式截然不同的是，代码兼容性涉及到随着时间的推移，使用 XCM 堆栈的 Rust 实现的（基于 Substrate 的）项目的代码库必须发生什么。

显然，旨在使用不断发展的语言来表达想法的代码库必须与时俱进，并进行调整。我们已经有了语义版本管理（SemVer）系统，它有助于规定在特定的版本变化中可能发生的变化。不过，这在处理 API 和 ABI 时确实很有用，而在考虑整体数据格式或语言时就不那么有用了。值得庆幸的是，XCM 的设计对 SemVer 的需求很小。

我们知道，较新版本的 XCM 软件能够在新旧 XCM 消息之间进行翻译，以及翻译其内部数据类型，如位置和资产。它之所以能够做到这一点，是因为在 XCM 代码库中同时保留了几个版本的 XCM 语言。Rust 的模块系统使这一点变得微不足道，一个新的 XCM 版本只需对应一个新的 Rust 模块。如果我们回顾一下 Rust 对 `VersionedXcm` 数据类型的声明（就在本文的开头），它只是底层 `Xcm` 数据类型的每个具体版本的标记联合，每个版本都在自己的模块 `v0`、`v1`、`v2` 等中找到。

由于使用 XCM 及其数据类型的事务和 API 倾向于只使用新旧格式同样可构建的版本变体，所以最终的结果是代码库可以被更新以使用最新的 XCM 软件（在 Rust 中，这被称为 crate），而对其代码的改动很少或没有改动。升级 XCM crate 可以使网络更好地与其他类似的升级网络进行互操作，但升级网络使用的任何 XCM 语言片段都不需要在以后进行。

我希望这能有力地激励团队保持其 XCM 板块的更新，从而保持一切快速迭代和发展。

## 🏁 总结

我希望这篇文章能让你了解 XCM 的版本系统，以及如何在主权链的语言在网络之间以不同的速度和时间演变时，利用它来保持网络的沟通，并且不给维护其逻辑的开发人员团队带来大量的操作费用。

在下一篇文章中，我们将更深入地探讨 XCM 最有趣的部分之一：其执行模型和异常管理能力。
