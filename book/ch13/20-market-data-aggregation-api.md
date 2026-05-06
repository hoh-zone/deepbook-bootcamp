# ch13-20 行情 API 聚合服务

[返回本章](README.md)

## 先看数据问题

这里先从读模型需求开始。“行情 API 聚合服务”要回答的是链上事件如何变成可查询、可分页、可对账、可监控的读模型。

## 源码入口

- [crates/server/src/reader.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/reader.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/server.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/server.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/schema/migrations/2026-02-03-000000-0000_add_ticker_performance_indexes/up.sql](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/migrations/2026-02-03-000000-0000_add_ticker_performance_indexes/up.sql)：表结构、索引、Diesel schema 或迁移约束。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。

## 落到读模型里

行情聚合服务不直接修改链上状态，只读取 Server API 或本地 PostgreSQL，并生成适合前端的数据结构：ticker、K 线、深度、最近成交、资金费率或 Margin 利率。这个服务的核心是缓存策略和降级策略，而不是复杂业务逻辑。


行情聚合服务应把成交、价格、成交量和池子元数据压成前端需要的 ticker 响应。聚合层要声明时间窗口、刷新频率和缺失数据策略，避免 UI 把短期无成交误读成价格为零。

## 数据系统判断

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 动手检查

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
