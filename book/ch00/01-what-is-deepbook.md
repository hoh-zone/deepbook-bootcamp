# ch00-01 DeepBook 解决什么问题

[返回本章](README.md)

DeepBook 是 Sui 上的去中心化中央限价订单簿，也就是 CLOB。它要解决的问题不是“如何用一条固定曲线完成资产兑换”，而是“如何把买卖双方的限价意愿放到链上，以价格优先、时间优先的方式撮合成交”。

在 AMM 中，交易者面对的是资金池曲线。价格由储备量和公式决定，流动性提供者把资产放进池子，交易者用滑点和手续费换取即时成交。DeepBook 的核心不同：它保存买盘和卖盘，maker 可以挂出具体价格和数量，taker 可以吃掉现有挂单，订单簿状态、成交、取消、费用和结算都通过链上对象和事件表达。

DeepBook 对开发者的价值主要有四类。第一，它提供链上撮合基础设施，协议和钱包不用自己实现订单簿。第二，它提供可组合的交易入口，应用可以选择限价单、市价单、直接 swap、批量撤单、查询订单簿等不同能力。第三，它提供账户和资金抽象，`BalanceManager` 让一个账户在多个池之间复用余额、权限和交易身份。第四，它提供数据层，事件、Indexer 和 Server 可以支撑行情、K 线、成交、账户历史和风控面板。

本书把 DeepBook 当成一套开发平台，而不只是一个交易页面。对于现货交易，你要理解 `Pool`、`Book`、`State`、`Vault` 和 `BalanceManager`。对于应用开发，你要会用 SDK 构造交易、用 dry run 做交易前检查、用 Indexer 查询事件和历史数据。对于高级产品，你还要区分 DeepBookV3 spot、DeepBook Margin 和 DeepBook Predict：它们在同一个仓库中，但并不是同一个链上状态机。

一句话概括：DeepBook 是 Sui 生态中的链上订单簿和交易基础设施；DeepBookV3 是现货核心，Margin 和 Predict 是围绕交易、借贷、风险和定价扩展出的上层产品。

## 读源码时的第一条边界

先从 [README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/README.md) 和 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 建立产品模型，再进入 `book/`、`state/`、`vault/` 等内部模块。否则很容易在撮合细节中迷失，看不清某个函数属于用户交易、协议管理、费用治理还是查询辅助。

## 本节检查

- [ ] 能解释 DeepBook 和 AMM 的差异。
- [ ] 能说明 DeepBook 对钱包、交易聚合器、做市商和后端服务的价值。
- [ ] 能区分“现货核心”和“上层产品”。
