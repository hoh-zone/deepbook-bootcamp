# ch13-19 本地启动路径

[返回本章](README.md)

## 本节目标

- 能按 PostgreSQL、Indexer、Server、API 检查的顺序启动本地数据系统。
- 能定位本节涉及的 handler、schema、Server 模块和部署入口，并说明它们之间的数据流。
- 能把“本地启动路径”用于交易终端、行情、Margin 面板或机器人风控的真实查询设计。

## 源码关联

- [docker/deepbook-indexer/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-indexer/Dockerfile)：容器构建、入口脚本或部署边界。
- [docker/deepbook-server/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-server/Dockerfile)：REST API、数据库查询、健康检查或指标实现。
- [crates/indexer/src/main.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/main.rs)：本节示例或运行路径。
- [crates/server/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/README.md)：REST API、数据库查询、健康检查或指标实现。

## 正文

本地最小路径：

1. 启动 PostgreSQL。
2. 设置 `DATABASE_URL`。
3. 在 [MystenLabs/deepbookv3](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0) 运行 `cargo run --package deepbook-indexer -- --env testnet --packages deepbook`。
4. 启动 `deepbook-server`。
5. 调用 `/status`、`/get_pools`、`/trades/:pool_name` 验证数据流。


GitHub 源码应先启动 PostgreSQL 和 migration，再启动 Indexer 追 checkpoint，最后启动 Server 验证 `/status` 和一个真实查询。调试时固定网络、package id、数据库 URL 和起始 checkpoint，才能让不同 worker 的结果可复现。

## 开发要点

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 检查问题

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
