# 官方文档基线：本书如何对齐 Sui 文档

本书不是官方文档的改写版，也不是源码注释合集。官方文档给出产品定位、当前网络状态、公开对象和集成入口；本书在这个基线上继续做三件事：解释源码为什么这样设计，训练 Move 协议阅读能力，并把交易、SDK、Indexer、风控和部署串成可落地的应用路径。

本页按 2026-05-06 访问到的 Sui 官方文档整理，后续写作和事实核对都应回到这些页面确认。

## DeepBookV3 的官方基线

官方定位：DeepBookV3 是构建在 Sui 上的去中心化中央限价订单簿。它利用 Sui 的并行执行和低交易费用，把高性能、低延迟的交易能力放到链上。官方还明确说明，DeepBookV3 本身不是面向终端用户的交易界面，而是为 DEX、钱包和其他应用提供内置交易能力。

本书采用的写法：

- 先用订单簿产品语言解释 bid、ask、limit、market、maker、taker。
- 再进入 `Pool -> Book -> State -> Vault -> BalanceManager` 的源码路径。
- SDK 和 Indexer 不写成附录，而写成应用开发主线：SDK 构造交易，Indexer/Server 支撑行情、历史和风控。
- DEEP tokenomics、staking、governance、fee discount 和 maker rebate 只在能落到源码或交易流程时展开。

官方入口：

- [DeepBookV3 overview](https://docs.sui.io/onchain-finance/deepbookv3/deepbook)
- [DeepBookV3 design](https://docs.sui.io/onchain-finance/deepbookv3/design)
- [DeepBookV3 contract information](https://docs.sui.io/onchain-finance/deepbookv3/contract-information)
- [DeepBookV3 SDK](https://docs.sui.io/onchain-finance/deepbookv3-sdk/)
- [DeepBookV3 Indexer](https://docs.sui.io/onchain-finance/deepbookv3/deepbookv3-indexer)

## DeepBook Margin 的官方基线

官方定位：DeepBook Margin 扩展 DeepBookV3 的交易能力，让用户通过借入资金建立杠杆仓位。官方文档的中心不是“多一个下单接口”，而是风险管理：抵押、借款、利息、oracle、风险率、清算和 TPSL 都是产品的一部分。

本书采用的写法：

- 先解释用户路径：存抵押、借 base 或 quote、通过 DeepBook Pool 交易、还款或被清算。
- 再解释对象关系：`MarginPool` 提供借贷流动性，`MarginManager` 包装 `BalanceManager`，`MarginRegistry` 管理池、版本、风险参数和启用状态。
- 利息和清算按“可被用户观察的风险”来写，而不是只讲公式。
- 每个 Margin 入口都必须回答：这一步增加风险还是降低风险，是否需要 oracle，是否允许 reduce-only，失败时归因到哪一层。

官方入口：

- [DeepBook Margin overview](https://docs.sui.io/onchain-finance/deepbook-margin/)
- [DeepBook Margin design](https://docs.sui.io/onchain-finance/deepbook-margin/design)
- [DeepBook Margin risks](https://docs.sui.io/onchain-finance/deepbook-margin/margin-risks)
- [DeepBook Margin contract information](https://docs.sui.io/onchain-finance/deepbook-margin/contract-information)

## DeepBook Predict 的官方基线

官方定位：DeepBook Predict 是 Sui 上基于到期日的预测市场协议。它支持 binary positions、vertical ranges、oracle-based pricing、`PredictManager` 共享账户、Vault liquidity 和 `PLP` LP shares。

官方文档同时强调当前状态：Predict 是 Testnet integration surface，智能合约在 Mainnet 部署前可能变化。应用不应把旧 package id、旧对象布局或实验脚本当作稳定生产接口。

本书采用的写法：

- Predict 章节必须持续标注网络和版本边界。
- 产品模型按 oracle、expiry、strike、range、vault、PLP、settlement 展开，不套用 Spot 订单簿模型。
- 应用集成采用三层数据模型：public Predict server 用于页面渲染和历史数据；checkpoint/event streaming 用于低延迟 oracle 状态；关键钱包流程前后再做直接 onchain object reads。
- `RangeKey`、`MarketKey`、`OracleSVI`、`PredictManager` 和 `Vault` 是本章的核心定义，不应被 UI 字符串替代。

官方入口：

- [DeepBook Predict overview](https://docs.sui.io/onchain-finance/deepbook-predict/)
- [DeepBook Predict design](https://docs.sui.io/onchain-finance/deepbook-predict/design)
- [DeepBook Predict contract information](https://docs.sui.io/onchain-finance/deepbook-predict/contract-information)

## 本书的出版级改写原则

官方文档适合快速集成和查对象。本书的章节必须在官方文档之上增加以下层次：

1. **读者问题**：这一节解决的是交易、风控、数据、SDK 还是 Move 安全问题。
2. **官方定位**：用官方文档确认能力边界、网络状态和公开集成入口。
3. **源码定义**：把关键 `struct`、`public fun`、事件或 handler 放进正文，而不是只给链接。
4. **状态路径**：说明对象如何被借用，余额如何移动，事件如何产生，失败如何 abort。
5. **工程落地**：把结论落实到 SDK、PTB、dry run、Indexer、UI、监控或部署。
6. **风险边界**：明确哪些能力是 Mainnet，哪些是 Testnet，哪些来自本书锁定源码快照，哪些仍需读官方最新文档确认。

如果一个小节只是在复述官方文档，它还不是书稿；如果一个小节只是在贴源码，它也还不是书稿。合格的小节应该让读者读完后，既知道官方能力边界，也能独立解释源码和实现一个可靠应用。
