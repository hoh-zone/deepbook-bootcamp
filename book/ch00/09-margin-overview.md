# ch00-09 DeepBook Margin 是什么

[返回本章](README.md)

DeepBook Margin 是围绕 DeepBook 现货池构建的杠杆交易和借贷系统。它不是把 `Pool` 改成“带杠杆模式”，而是单独放在 [packages/deepbook_margin/sources](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources) 中。

官方文档给 Margin 的第一层定位是“用借入资金放大交易能力”，但它的真正主线是风险管理。一个 Margin 应用如果只展示杠杆倍数，而不展示风险率、利率、清算阈值和 oracle freshness，就没有把协议讲完整。

Margin 的核心对象是 `MarginManager`。源码中的 [margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move) 显示，它包装了一个 `BalanceManager`，并保存对应的 `DepositCap`、`WithdrawCap`、`TradeCap`、借款 shares、deepbook pool id 和 TPSL 状态。也就是说，Margin 不是绕过 DeepBook 账户模型，而是把它作为交易账户底座。

Margin 还依赖借贷池、oracle、风险参数和清算逻辑。`margin_pool.move` 管供应、借款和利息；`oracle.move` 处理价格；`margin_registry.move` 管注册和配置；`pool_proxy.move` 负责把 Margin 交易接入 DeepBook 池；`tpsl.move` 提供 take profit / stop loss 条件订单。

从用户体验看，Margin 提供抵押、借款、杠杆买入、杠杆卖出、还款、平仓、清算和条件订单。从开发者角度看，更重要的是每个动作都需要风险检查：价格是否新鲜、风险率是否超过阈值、借款资产是否匹配、DeepBook pool 是否允许 Margin 交易、还款数量是否足够、清算者是否能获得奖励。

本书后续会把 Margin 分成协议源码和应用开发两章。第 08 章读核心 Move 状态机，第 09 章把它落到 SDK、PTB、CLI、清算机器人和风控面板中。

官方入口：

- [DeepBook Margin overview](https://docs.sui.io/onchain-finance/deepbook-margin/)
- [DeepBook Margin design](https://docs.sui.io/onchain-finance/deepbook-margin/design)
- [DeepBook Margin risks](https://docs.sui.io/onchain-finance/deepbook-margin/margin-risks)

## 本节检查

- [ ] 能解释 `MarginManager` 与 `BalanceManager` 的关系。
- [ ] 能列出 Margin 依赖的模块：借贷池、oracle、registry、pool proxy、TPSL。
- [ ] 能区分 Margin 借贷和现货闪电贷。
- [ ] 能用风险率、利息和清算解释为什么 Margin 不是普通 spot 下单。
