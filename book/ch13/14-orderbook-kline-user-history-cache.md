# ch13-14 订单簿缓存、K 线与用户历史

[返回本章](README.md)

## 本节目标

- 理解订单簿缓存、K 线聚合和用户历史查询的不同数据刷新策略。
- 能定位本节涉及的 handler、schema、Server 模块和部署入口，并说明它们之间的数据流。
- 能把“订单簿缓存、K 线与用户历史”用于交易终端、行情、Margin 面板或机器人风控的真实查询设计。

## 源码关联

- [crates/server/src/reader.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/reader.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/schema/migrations/2025-09-30-202714-0000_add_ohclv/up.sql](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/migrations/2025-09-30-202714-0000_add_ohclv/up.sql)：表结构、索引、Diesel schema 或迁移约束。
- [crates/schema/migrations/2025-10-13-194059-0000_fix_ohclv_price_calculation/up.sql](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/migrations/2025-10-13-194059-0000_fix_ohclv_price_calculation/up.sql)：表结构、索引、Diesel schema 或迁移约束。
- [crates/indexer/src/handlers/order_fill_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_fill_handler.rs)：事件解析、字段映射或业务流水落库入口。

## 正文

订单簿实时展示可以结合链上 level2 调用和本地缓存。K 线不应每次请求都扫全表，应该按固定周期预聚合或使用物化视图。用户历史订单需要同时合并 `order_updates` 和 `order_fills`，因为订单更新描述状态，成交表描述实际成交。


订单簿快照、成交流、K 线和用户历史来自不同粒度的数据。`order_fills` 适合生成成交流和 OHLCV，订单簿实时层更适合短 TTL 缓存；用户历史必须按 BalanceManager 或地址过滤，避免把全市场成交当作用户成交。

## 开发要点

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 检查问题

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
