# ch09-15 实战：Margin 风控面板

[返回本章](README.md)

## 先看用户路径

这一节从 Margin 用户动作读起：围绕“实战：Margin 风控面板”，先把 manager、collateral、loan、trade 和 repay 的顺序排清楚，再看源码入口。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_registry.move)
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/margin_pool/margin_state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/margin_state.move)
- [packages/margin_liquidation/sources/liquidation_vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/margin_liquidation/sources/liquidation_vault.move)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 把 PTB 串起来

风控面板可以由三张表和一个详情页组成：

- Pool 表：pool key、enabled、base/quote MarginPool、risk params、当前价格、价格更新时间。
- Lending 表：asset、total supply、total borrow、utilization、borrow rate、supply cap、vault balance。
- Manager 表：owner、manager ID、debt asset、risk ratio、debt amount、base/quote asset、TPSL 数量。
- 详情页：订单、成交、抵押事件、借还事件、清算事件和 dry run 面板。

后台同步优先消费事件：

- `MarginManagerCreatedEvent`
- `DepositCollateralEvent`
- `WithdrawCollateralEvent`
- `LoanBorrowedEvent`
- `LoanRepaidEvent`
- `LiquidationEvent`
- TPSL 相关事件
- `AssetSupplied`、`AssetWithdrawn`

风险率要定时用最新 Pyth 价格刷新。事件只能告诉你状态发生过变化，不能替代当前价格下的健康度计算。

## 工程旁白

风控面板不是纯事件列表。事件用于发现状态变化，但风险率、利息和可清算状态必须用最新 price object 与 pool state 重新计算；否则价格剧烈变化时，面板会显示过期健康度。

运营视角需要同时看借贷池和交易账户：某个池 utilization 过高会影响新增借款和提款，某个 manager 风险率过低会影响清算队列，vault 库存不足会影响机器人执行率。

## Margin 应用判断

- 数据层按 pool key 聚合 registry、MarginPool、manager 和 oracle 信息。
- 面板标注数据来源和刷新时间，尤其是 risk ratio 与 price timestamp。
- 清算队列表包含 vault 库存检查和最近一次 dev inspect 结果。

## 动手检查

- 面板展示的是实时链上读取、定时刷新结果还是 indexer 缓存？
- 哪个池的 utilization 或 stale price 正在影响最多用户操作？
- 清算候选是否同时满足风险率、vault 库存和预期收益条件？
