# ch13-15 Docker 部署

[返回本章](README.md)

## 先看数据问题

这一节从读模型而不是数据库表开始。围绕“Docker 部署”，先问交易终端、机器人或风控系统要查什么，再看 handler 和 schema 如何支撑。

## 源码入口

- [docker/deepbook-indexer/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-indexer/Dockerfile)：容器构建、入口脚本或部署边界。
- [docker/deepbook-server/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-server/Dockerfile)：REST API、数据库查询、健康检查或指标实现。
- [docker/deepbook-server/entry.sh](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-server/entry.sh)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/main.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/main.rs)：REST API、数据库查询、健康检查或指标实现。

## 落到读模型里

生产拓扑通常包含 PostgreSQL、Indexer、Server、Prometheus、Grafana 和前端。Indexer 只负责写库，Server 只负责读库和链上只读调用，两者应独立扩缩容。数据库是关键状态，不要让应用容器启动脚本隐式清空数据。


容器部署时 Indexer、Server 和 PostgreSQL 是三个不同故障域。Indexer 需要稳定 RPC 和写库权限，Server 需要只读连接池和 metrics 端口，迁移最好作为独立步骤执行，避免服务启动并发抢锁。

## 数据系统判断

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 动手检查

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
