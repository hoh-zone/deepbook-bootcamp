# ch14-03 订单簿和成交流组件

[返回本章](README.md)

## 先定义产品场景

产品小节先想用户路径和失败路径。DeepBook 应用的难点往往不在单个 move call，而在状态同步、错误解释和风险降级。

## 源码入口

- [crates/server/src/reader.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/reader.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/indexer/src/handlers/order_fill_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_fill_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/schema/migrations/2025-09-30-202714-0000_add_ohclv/up.sql](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/migrations/2025-09-30-202714-0000_add_ohclv/up.sql)：表结构、索引、Diesel schema 或迁移约束。
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。

## 把系统拼起来

订单簿展示 bid/ask 价格、数量和累计深度。成交流展示价格、数量、方向和时间。数据来自 Server API 或自建 Indexer。需要处理数据延迟、空数据、价格精度和刷新节流。


订单簿组件需要合并实时快照、增量刷新和成交流。成交流来自 `order_fills`，K 线来自 OHLCV 聚合；价格档位的展示则要有深度、精度和更新时间，不能只渲染最近成交价。

## 产品落地判断

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 动手检查

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
