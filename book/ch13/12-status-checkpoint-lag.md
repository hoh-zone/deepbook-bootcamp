# ch13-12 `/status` 与 checkpoint lag

[返回本章](README.md)

## 先看数据问题

这里先从读模型需求开始。“/status 与 checkpoint lag”要回答的是链上事件如何变成可查询、可分页、可对账、可监控的读模型。

## 源码入口

- [crates/indexer/src/handlers/mod.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/mod.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。
- [crates/indexer/tests/snapshot_tests.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/snapshot_tests.rs)：测试 fixture、断言或可复现验证材料。
- [crates/indexer/tests/checkpoints/order_fill/100000337.chk](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/checkpoints/order_fill/100000337.chk)：测试 fixture、断言或可复现验证材料。

## 落到读模型里

`/status` 返回每个 pipeline 的 indexed checkpoint、latest on-chain checkpoint、checkpoint lag 和 time lag。风控系统至少要监控：

- `status` 是否为 `OK`。
- `max_checkpoint_lag` 是否超过阈值。
- `max_time_lag_seconds` 是否超过阈值。
- `max_lag_pipeline` 是哪个 pipeline。

当 Indexer 落后时，前端应提示数据延迟，自动交易系统应降低频率或暂停依赖历史数据的策略。


`EventMeta` 把 checkpoint、高度时间戳、交易 digest 和 event digest 一起传给 handler。表以 `event_digest` 作为幂等键，并保留 `checkpoint`、`checkpoint_timestamp_ms`，这样重放 fixture、排查交易回执和 API 时间窗口查询可以使用同一组定位字段。

## 数据系统判断

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 动手检查

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
