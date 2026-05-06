# ch13 DeepBook Indexer、Server 与数据系统

## 本章目标

官方 DeepBookV3 Indexer 文档把 Indexer 定位为实时访问订单簿和交易数据的链下服务。它既提供 public service，也允许应用自建服务；选择哪一种，取决于延迟、可用性、定制需求和运维能力。

本章目标是把这个官方定位落成工程系统：理解为什么交易应用不能只依赖链上查询；掌握 `crates/indexer` 从 checkpoint 读取事件、解析事件、写入 PostgreSQL 的路径；掌握 `crates/server` 如何把数据库和链上只读调用封装为 REST API；最终能为交易终端、行情页、Margin 面板和风控系统设计可维护的数据服务。

## 官方数据基线

| 官方能力 | 本章展开 |
| --- | --- |
| Public DeepBookV3 indexer | 解释何时可以直接用 public endpoint，何时必须自建。 |
| `/get_pools`、historical volume、OHLCV、user volume | 映射到池元数据、K 线、个人交易历史和资产精度处理。 |
| Asset conversions | 所有返回 volume 都要用资产 decimals/scalar 解释，不能直接显示整数。 |
| Margin events | 区分 DeepBook flashloans、Margin loan borrowed/repaid、liquidation 等不同生命周期。 |
| `/status` 与服务健康 | 机器人、做市和风控系统必须根据 checkpoint lag 降级或暂停。 |

## 本章学习阶梯

- L1 先理解为什么交易应用不能只靠实时链上查询。
- L2 读 checkpoint、event digest、handler、schema 和 Server route。
- L3 设计订单、成交、闪电贷、Margin 事件的表和 API。
- L5 处理 checkpoint lag、缓存一致性、部署和监控。

## 关键定义卡片

Indexer 的核心不是“查对象”，而是把事件落成读模型。以成交事件为例：

```rust
define_handler! {
    name: OrderFillHandler,
    processor_name: "order_fill",
    event_type: OrderFilled,
    db_model: OrderFill,
    table: order_fills,
    map_event: |event, meta| OrderFill {
        event_digest: meta.event_digest(),
        digest: meta.digest(),
        checkpoint: meta.checkpoint(),
        pool_id: event.pool_id.to_string(),
        maker_order_id: event.maker_order_id.to_string(),
        taker_order_id: event.taker_order_id.to_string(),
        price: event.price as i64,
        base_quantity: event.base_quantity as i64,
        quote_quantity: event.quote_quantity as i64,
        onchain_timestamp: event.timestamp as i64,
    }
}
```

这段定义告诉我们四件事：事件类型是 `OrderFilled`，目标表是 `order_fills`，主键语义来自 `event_digest`，时间窗口查询依赖 checkpoint 和 onchain timestamp。前端成交历史、K 线和账户流水都应该从事件语义设计，而不是从当前对象状态反推。

## 源码地图

- [Sui Docs: DeepBookV3 Indexer](https://docs.sui.io/onchain-finance/deepbookv3/deepbookv3-indexer)：public indexer、asset conversion、API endpoints、自建服务取舍。
- [crates/indexer/src/main.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/main.rs)：Indexer 启动参数、环境选择、pipeline 注册。
- [crates/indexer/src/handlers/mod.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/mod.rs)：`EventMeta`、`define_handler!` 宏、DeepBook 交易过滤。
- [crates/indexer/src/handlers/order_fill_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_fill_handler.rs)：成交事件到 `order_fills` 表的映射。
- [crates/indexer/src/handlers/flash_loan_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/flash_loan_handler.rs)：闪电贷事件到 `flashloans` 表的映射。
- [crates/schema/migrations/2025-03-19-104023_deepbook/up.sql](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/migrations/2025-03-19-104023_deepbook/up.sql)：核心 DeepBook 表结构。
- [crates/server/src/server.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/server.rs)：REST API 路由、状态检查、行情和 Margin 端点。
- [docker/deepbook-indexer/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-indexer/Dockerfile)：Indexer 容器构建。
- [docker/deepbook-server/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-server/Dockerfile)：Server 容器构建。

## 小节目录

- [01 为什么交易应用不能只依赖链上查询](01-why-offchain-data.md)
- [02 Sui checkpoint、event 与 digest 模型](02-checkpoint-event-digest-model.md)
- [03 `sui-indexer-alt` 在本项目中的使用方式](03-sui-indexer-alt-pipelines.md)
- [04 Indexer 启动参数](04-indexer-startup-args.md)
- [05 Handler 与 `map_event` 模式](05-handler-map-event-pattern.md)
- [06 Core DeepBook handlers](06-core-deepbook-handlers.md)
- [07 Flash Loan handler 与 `flashloans` 表](07-flashloan-handler-table.md)
- [08 Margin handlers](08-margin-handlers.md)
- [09 Schema migrations 与 Diesel model](09-schema-migrations-diesel.md)
- [10 PostgreSQL 查询模式](10-postgres-query-patterns.md)
- [11 Server REST API](11-server-rest-api.md)
- [12 `/status` 与 checkpoint lag](12-status-checkpoint-lag.md)
- [13 Prometheus metrics](13-prometheus-metrics.md)
- [14 订单簿缓存、K 线与用户历史](14-orderbook-kline-user-history-cache.md)
- [15 Docker 部署](15-docker-deployment.md)
- [16 数据一致性](16-data-consistency.md)
- [17 前端如何消费 API](17-frontend-api-consumption.md)
- [18 自建 Indexer 与公共 API 的取舍](18-self-hosted-vs-public-api.md)
- [19 本地启动路径](19-local-startup-path.md)
- [20 行情 API 聚合服务](20-market-data-aggregation-api.md)

## 本章代码

- `code/s01-local-postgres/`：本地 PostgreSQL 启动和数据库准备。
- `code/s02-run-indexer/`：运行 DeepBook Indexer 的命令模板。
- `code/s03-query-events/`：查询订单、成交、闪电贷和 Margin 事件。
- `code/s04-kline-builder/`：从成交表生成 K 线的思路。
- `code/s05-api-client/`：调用 DeepBook Server API。
- `code/s06-monitoring/`：检查 `/status` 和 Prometheus 指标。

## Move 高阶穿插点

- Indexer 学习重点是事件语义：事件不是日志装饰，而是 Move 状态迁移留给链下系统的事实边界。
- 设计表结构时先确认事件是否唯一、是否可重放、是否需要 transaction digest 和 event index 组成幂等键。
- Server API 不应该伪造链上不存在的确定性，遇到延迟和重组风险要明确暴露数据时间。

## 常见错误

- 把 public indexer 当作无限 SLA。生产机器人和风控系统需要明确延迟阈值和降级策略。
- 把链上对象状态和 Indexer 落库状态视为完全同步。
- 把 DeepBookV3 闪电贷表当成 Margin 借贷表。
- 忘记给高频查询加时间窗口和分页。
- 在迁移中修改大表字段但没有评估锁表时间。
- Indexer 落后时继续运行依赖实时数据的机器人。

## 本章检查清单

- [ ] 能说清 checkpoint、event、digest、event_digest 的关系。
- [ ] 能找到 core DeepBook 和 Margin handlers 的注册位置。
- [ ] 能解释 `define_handler!` 如何把事件写入数据库。
- [ ] 能区分 `flashloans` 和 `loan_borrowed`。
- [ ] 能设计交易终端所需的最小查询表。
- [ ] 能使用 `/status` 判断 Indexer 是否健康。

## 进阶练习

- 为 `order_fills` 设计一个 1 分钟 K 线物化视图。
- 为 `flashloans` 写一个按池子统计每日借出数量的 SQL。
- 设计一个当 checkpoint lag 超过阈值时自动暂停做市机器人的机制。
