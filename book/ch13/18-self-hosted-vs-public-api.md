# ch13-18 自建 Indexer 与公共 API 的取舍

[返回本章](README.md)

## 先看数据问题

这里先从读模型需求开始。“自建 Indexer 与公共 API 的取舍”要回答的是链上事件如何变成可查询、可分页、可对账、可监控的读模型。

## 源码入口

- [crates/indexer/src/main.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/main.rs)：本节示例或运行路径。
- [crates/server/src/server.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/server.rs)：REST API、数据库查询、健康检查或指标实现。
- [docker/deepbook-indexer/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-indexer/Dockerfile)：容器构建、入口脚本或部署边界。
- [docker/deepbook-server/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-server/Dockerfile)：REST API、数据库查询、健康检查或指标实现。

## 落到读模型里

早期应用可以使用公共 API 快速开发，但专业交易、做市、清算和风控系统应自建 Indexer。自建的价值是可控延迟、可控表结构、可回放、可审计。代价是数据库运维、迁移管理、链上数据兼容和监控成本。


公共 API 适合低频查询和原型，自建 Indexer 适合机器人、风控和审计留痕。取舍点包括延迟、可用性、历史保留、查询自由度和故障责任；一旦应用需要按业务字段聚合或回放，就应准备自建数据层。

## 数据系统判断

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 动手检查

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
