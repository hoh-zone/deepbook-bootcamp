# ch13-03 `sui-indexer-alt` 在本项目中的使用方式

[返回本章](README.md)

## 本节目标

- 理解 `sui-indexer-alt` 如何按 pipeline 消费 checkpoint 并并发写入数据库。
- 能定位本节涉及的 handler、schema、Server 模块和部署入口，并说明它们之间的数据流。
- 能把“`sui-indexer-alt` 在本项目中的使用方式”用于交易终端、行情、Margin 面板或机器人风控的真实查询设计。

## 源码关联

- [crates/indexer/src/main.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/main.rs)：本节示例或运行路径。
- [crates/indexer/src/handlers/mod.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/mod.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/indexer/tests/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/README.md)：测试 fixture、断言或可复现验证材料。
- [docker/deepbook-indexer/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-indexer/Dockerfile)：容器构建、入口脚本或部署边界。

## 正文

`crates/indexer/src/main.rs` 使用 `Indexer::new(...)` 创建索引器，然后对每类事件注册 `concurrent_pipeline`。核心包由 `--packages deepbook` 控制，Margin 包由 `--packages deepbook-margin` 控制。

生产模式要求传入 `--env testnet|mainnet`。Sandbox 模式允许指定自定义 package id 和本地 checkpoint 路径，适合本地部署 DeepBook 后做集成测试。


本项目把 `sui-indexer-alt` 当作 checkpoint 消费框架使用，业务逻辑集中在 handlers。新增事件时通常先在 `handlers/mod.rs` 注册 handler，再补 Diesel model 和 migration，最后用 checkpoint fixture 做 snapshot，避免 pipeline 能启动但字段落库错误。

## 开发要点

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 检查问题

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
