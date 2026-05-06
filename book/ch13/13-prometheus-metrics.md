# ch13-13 Prometheus metrics

[返回本章](README.md)

## 本节目标

- 能识别 Indexer、Server 和数据库需要暴露给 Prometheus 的关键指标。
- 能定位本节涉及的 handler、schema、Server 模块和部署入口，并说明它们之间的数据流。
- 能把“Prometheus metrics”用于交易终端、行情、Margin 面板或机器人风控的真实查询设计。

## 源码关联

- [crates/server/src/metrics/mod.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/metrics/mod.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/metrics/middleware.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/metrics/middleware.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/margin_metrics/metrics.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/margin_metrics/metrics.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/margin_metrics/poller.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/margin_metrics/poller.rs)：REST API、数据库查询、健康检查或指标实现。

## 正文

Indexer 默认暴露 Prometheus 指标。需要纳入生产告警的指标包括数据库连接、pipeline 处理延迟、checkpoint lag、错误数、Server 请求延迟和 HTTP 5xx。指标只是告警入口，排障还需要保留 digest、checkpoint、pipeline name 和数据库写入错误。


Prometheus 指标要覆盖 API 延迟、数据库请求成功率、Margin 轮询错误和业务派生值。告警阈值应结合 `/status` 使用：单次 RPC 慢不一定影响交易，但 checkpoint lag 扩大说明读模型已不适合驱动机器人。

## 开发要点

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 检查问题

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
