# ch13-11 Server REST API

[返回本章](README.md)

## 本节目标

- 熟悉 DeepBook Server 暴露的行情、订单、池子和 Margin API 边界。
- 能定位本节涉及的 handler、schema、Server 模块和部署入口，并说明它们之间的数据流。
- 能把“Server REST API”用于交易终端、行情、Margin 面板或机器人风控的真实查询设计。

## 源码关联

- [crates/server/src/server.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/server.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/reader.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/reader.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/writer.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/writer.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/README.md)：REST API、数据库查询、健康检查或指标实现。

## 正文

`crates/server/src/server.rs` 暴露 DeepBook 和 Margin 数据 API。核心路径包括：

- `/get_pools`：池子列表。
- `/ticker`：行情摘要。
- `/trades/:pool_name`：成交记录。
- `/order_updates/:pool_name`：订单更新。
- `/orders/:pool_name/:balance_manager_id`：用户订单。
- `/orderbook/:pool_name`：二级订单簿。
- `/ohclv/:pool_name`：K 线。
- `/loan_borrowed`、`/loan_repaid`、`/liquidation`：Margin 事件。
- `/status`：Indexer 健康状态。

Server 同时会做链上只读调用，例如订单簿 level2、费用参数、DEEP supply。应用要区分数据库读模型和链上实时读模型的延迟差异。


REST 层把数据库读模型、链上只读 RPC 和管理端点拆开。行情、历史成交和 Margin metrics 走只读 reader；需要触发写操作或管理行为时应经过明确鉴权、限流和错误码映射，避免前端把内部错误直接展示成交易失败原因。

## 开发要点

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 检查问题

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
