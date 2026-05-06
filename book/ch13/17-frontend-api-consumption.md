# ch13-17 前端如何消费 API

[返回本章](README.md)

## 先看数据问题

这一节从读模型而不是数据库表开始。围绕“前端如何消费 API”，先问交易终端、机器人或风控系统要查什么，再看 handler 和 schema 如何支撑。

## 源码入口

- [crates/server/src/server.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/server.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/README.md)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/reader.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/reader.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。

## 落到读模型里

前端应把 API 分成三类：

- 快速刷新：ticker、orderbook、recent trades。
- 用户相关：orders、fills、balances、Margin positions。
- 低频配置：pools、fees、assets、risk params。

每类使用不同缓存 TTL。用户下单后的状态不应只靠轮询，要结合交易 digest 查询、Indexer API 和链上对象状态做确认。


前端消费 API 要把“交易已上链”和“读模型已刷新”拆成两个状态。提交 PTB 后先展示 digest 和链上确认，再轮询历史成交、订单更新或 `/status`；当 checkpoint lag 超阈值时，应提示数据延迟而不是重复发交易。

## 数据系统判断

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 动手检查

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
