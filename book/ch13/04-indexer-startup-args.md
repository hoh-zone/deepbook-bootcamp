# ch13-04 Indexer 启动参数

[返回本章](README.md)

## 先看数据问题

这里先从读模型需求开始。“Indexer 启动参数”要回答的是链上事件如何变成可查询、可分页、可对账、可监控的读模型。

## 源码入口

- [crates/indexer/src/main.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/main.rs)：本节示例或运行路径。
- [docker/deepbook-indexer/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-indexer/Dockerfile)：容器构建、入口脚本或部署边界。
- [crates/server/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/README.md)：REST API、数据库查询、健康检查或指标实现。

## 落到读模型里

默认数据库地址来自 `database_url` 参数，默认值是 `postgres://postgres:postgrespw@localhost:5432/deepbook`。常用启动方式：

```bash
DATABASE_URL="postgresql://postgres:postgrespw@localhost:5432/deepbook" \
cargo run --package deepbook-indexer -- --env testnet --packages deepbook deepbook-margin
```

关键参数：

- `--env`：选择 `testnet` 或 `mainnet`。
- `--packages`：选择 `deepbook`、`deepbook-margin` 或两者同时启用。
- `--metrics-address`：Prometheus 指标地址，默认 `0.0.0.0:9184`。
- `sandbox` 子命令：用本地或测试网自定义包进行索引。


启动参数决定网络、数据库连接、起始 checkpoint 和 pipeline 组合。生产环境要把 RPC、PostgreSQL、metrics 端口和重放起点显式写入部署清单，避免容器重启后从错误网络或错误 checkpoint 继续消费。

## 数据系统判断

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 动手检查

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
