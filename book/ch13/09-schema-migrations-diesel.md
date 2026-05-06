# ch13-09 Schema migrations 与 Diesel model

[返回本章](README.md)

## 本节目标

- 理解 schema migration、Diesel model 和 API 响应之间的版本耦合。
- 能定位本节涉及的 handler、schema、Server 模块和部署入口，并说明它们之间的数据流。
- 能把“Schema migrations 与 Diesel model”用于交易终端、行情、Margin 面板或机器人风控的真实查询设计。

## 源码关联

- [crates/schema/migrations/2025-03-19-104023_deepbook/up.sql](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/migrations/2025-03-19-104023_deepbook/up.sql)：表结构、索引、Diesel schema 或迁移约束。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。
- [crates/schema/src/models.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/models.rs)：表结构、索引、Diesel schema 或迁移约束。
- [crates/schema/migrations/2025-10-24-000000-0000_add_margin_table_indexes/up.sql](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/migrations/2025-10-24-000000-0000_add_margin_table_indexes/up.sql)：表结构、索引、Diesel schema 或迁移约束。

## 正文

`crates/schema/migrations` 是数据库事实来源。初始迁移创建了 `order_updates`、`order_fills`、`flashloans`、`pool_prices`、`balances`、`trade_params_update` 等表。后续迁移增加 Margin 表、OHCLV、索引、返佣 v2、池子快照等。

生产上必须把 migrations 当成协议兼容层管理：事件字段变化、表字段变化、API 响应变化要一起评估。Indexer 启动时会执行 pending migrations，因此部署新版本前要先验证迁移可重复执行、可回滚、不会锁表过久。


迁移文件定义数据库事实，`schema.rs` 和 `models.rs` 则让 Indexer/Server 在编译期使用同一套字段。修改大表时要同步考虑主键、唯一约束、索引和回填成本，尤其是 `event_digest` 幂等键与按 `checkpoint_timestamp_ms` 查询的复合索引。

## 开发要点

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 检查问题

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
