# ch13-01 为什么交易应用不能只依赖链上查询

[返回本章](README.md)

## 先看数据问题

这一节从读模型而不是数据库表开始。围绕“为什么交易应用不能只依赖链上查询”，先问交易终端、机器人或风控系统要查什么，再看 handler 和 schema 如何支撑。

## 源码入口

- [crates/indexer/src/main.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/main.rs)：Indexer 启动、参数解析和 handler 装载入口。
- [crates/indexer/src/handlers/order_fill_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_fill_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。
- [crates/server/src/reader.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/reader.rs)：REST API、数据库查询、健康检查或指标实现。

## 落到读模型里

DeepBook 的链上对象适合保存最终可信状态，但交易产品还需要低延迟、可分页、可聚合的数据。订单簿页面需要最近成交、K 线、用户历史订单、订单状态、手续费、池子列表和 Margin 风险数据。如果每次都从 RPC 扫对象和事件，前端会面对三个问题：查询慢、分页困难、历史聚合成本高。

链下数据层的职责不是替代链上状态，而是把链上事件转成应用可查询的读模型。交易提交和资金安全仍由 Move 合约保证；Indexer 负责把 `OrderFilled`、`OrderUpdate`、`FlashLoanBorrowed`、`LoanBorrowed`、`Liquidation` 等事件落库。


交易终端需要的是可分页、可过滤、可聚合的读模型，而不是一次次扫描链上对象。Indexer 从 checkpoint 抽取成交、订单更新、余额和池子事件后写入 PostgreSQL，Server 再用 `reader.rs` 把 `order_fills`、`balances`、`pool_prices` 等表裁剪成前端 API。这里的核心边界是：链上对象仍是最终可信状态，数据库只是为查询体验和监控服务。

## 数据系统判断

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 动手检查

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
