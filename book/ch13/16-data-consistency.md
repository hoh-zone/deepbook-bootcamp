# ch13-16 数据一致性

[返回本章](README.md)

## 本节目标

- 掌握事件幂等写入、checkpoint 重放和读模型延迟带来的一致性边界。
- 能定位本节涉及的 handler、schema、Server 模块和部署入口，并说明它们之间的数据流。
- 能把“数据一致性”用于交易终端、行情、Margin 面板或机器人风控的真实查询设计。

## 源码关联

- [crates/indexer/src/handlers/mod.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/mod.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。
- [crates/server/src/server.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/server.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/indexer/tests/snapshot_tests.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/snapshot_tests.rs)：测试 fixture、断言或可复现验证材料。

## 正文

事件写入必须幂等。当前 handler 使用 `event_digest` 作为主键，并 `on_conflict_do_nothing`，这允许 pipeline 重放 checkpoint 而不重复写入。需要注意的是：如果事件映射逻辑变更，旧数据不会自动重算，必须设计数据回填或重建流程。


数据一致性依赖幂等写入、checkpoint 顺序和可观测的落后程度。API 返回的数据应携带或内部记录 checkpoint 范围，前端确认交易时用 digest 对齐链上执行结果，再等待 Indexer 追上对应 checkpoint。

## 开发要点

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 检查问题

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
