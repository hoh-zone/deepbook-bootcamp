# ch14-03 订单簿和成交流组件

[返回本章](README.md)

## 本节目标

- 明确“订单簿组件”在 DeepBook 应用闭环中的用户场景、交易构造和数据依赖。
- 能把 UI、SDK/PTB、Server API、Indexer 数据和监控信号连接成可运行流程。
- 能识别本节应用上线前必须处理的余额、风险、延迟、失败和披露问题。

## 源码关联

- [crates/server/src/reader.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/reader.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/indexer/src/handlers/order_fill_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_fill_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/schema/migrations/2025-09-30-202714-0000_add_ohclv/up.sql](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/migrations/2025-09-30-202714-0000_add_ohclv/up.sql)：表结构、索引、Diesel schema 或迁移约束。
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。

## 正文

订单簿展示 bid/ask 价格、数量和累计深度。成交流展示价格、数量、方向和时间。数据来自 Server API 或自建 Indexer。需要处理数据延迟、空数据、价格精度和刷新节流。


订单簿组件需要合并实时快照、增量刷新和成交流。成交流来自 `order_fills`，K 线来自 OHLCV 聚合；价格档位的展示则要有深度、精度和更新时间，不能只渲染最近成交价。

## 开发要点

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 检查问题

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
