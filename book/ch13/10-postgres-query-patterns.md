# ch13-10 PostgreSQL 查询模式

[返回本章](README.md)

## 先看数据问题

这里先从读模型需求开始。“PostgreSQL 查询模式”要回答的是链上事件如何变成可查询、可分页、可对账、可监控的读模型。

## 源码入口

- [crates/server/src/reader.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/reader.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/schema/migrations/2025-09-30-200612-0000_add_indexes/up.sql](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/migrations/2025-09-30-200612-0000_add_indexes/up.sql)：表结构、索引、Diesel schema 或迁移约束。
- [crates/schema/migrations/2026-02-10-000000-0000_add_orders_endpoint_indexes/up.sql](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/migrations/2026-02-10-000000-0000_add_orders_endpoint_indexes/up.sql)：表结构、索引、Diesel schema 或迁移约束。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。

## 落到读模型里

常见查询模式：

- 最近成交：按 `pool_id` 和 `checkpoint_timestamp_ms` 倒序查 `order_fills`。
- K 线：按时间桶聚合 `order_fills` 的 price、base quantity、quote quantity。
- 用户订单：按 `balance_manager_id` 查 `order_updates`。
- 用户成交：按 maker 或 taker `balance_manager_id` 查 `order_fills`。
- 闪电贷统计：按 `pool_id`、`type_name` 聚合 `flashloans`。
- Margin 借贷：按 `margin_manager_id` 查 `loan_borrowed` 和 `loan_repaid`。

这些查询都应有复合索引支撑。读多写多的表要避免前端任意排序，API 应限制时间窗口和分页大小。


Server 的高频查询都应限制池子、账户和时间窗口。成交流、用户历史和订单列表不能开放无界扫描；分页字段最好与索引顺序一致，例如按 `checkpoint_timestamp_ms` 倒序、再用 `event_digest` 或订单 id 做稳定游标。

## 数据系统判断

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 动手检查

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
