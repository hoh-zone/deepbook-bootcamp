# ch13-06 Core DeepBook handlers

[返回本章](README.md)

## 本节目标

- 能列出核心 DeepBook handlers，并说明它们分别支撑订单、成交、余额和池子查询。
- 能定位本节涉及的 handler、schema、Server 模块和部署入口，并说明它们之间的数据流。
- 能把“Core DeepBook handlers”用于交易终端、行情、Margin 面板或机器人风控的真实查询设计。

## 源码关联

- [crates/indexer/src/handlers/order_fill_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_fill_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/indexer/src/handlers/order_update_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_update_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/indexer/src/handlers/balances_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/balances_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/indexer/src/handlers/pool_price_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/pool_price_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。

## 正文

核心 DeepBook pipeline 包括：

- `OrderFillHandler`：写入 `order_fills`。
- `OrderUpdateHandler`：写入 `order_updates`，用于订单状态。
- `BalancesHandler`：写入 `balances`，用于 BalanceManager 资金历史。
- `PoolCreatedHandler`：写入池子创建事件。
- `PoolPriceHandler`：写入参考价格。
- `TradeParamsUpdateHandler`：写入交易参数变化。
- `StakesHandler`、`RebatesHandler`、`RebatesV2Handler`：写入 DEEP stake 和返佣数据。
- `BookParamsUpdatedHandler`：写入 tick、lot 等 book 参数变化。

交易终端通常至少需要 `order_fills`、`order_updates`、`balances`、`pools` 和 `pool_prices`。


核心 handlers 覆盖撮合结果、订单状态、余额变化和池子价格。交易终端通常用 `order_fills` 生成成交流和 K 线，用 `order_updates` 跟踪订单生命周期，用 `balances` 辅助解释 BalanceManager 维度的资金变化。

## 开发要点

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 检查问题

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
