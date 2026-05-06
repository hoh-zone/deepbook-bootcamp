# ch14-12 风控系统

[返回本章](README.md)

## 本节目标

- 明确“风控系统”在 DeepBook 应用闭环中的用户场景、交易构造和数据依赖。
- 能把 UI、SDK/PTB、Server API、Indexer 数据和监控信号连接成可运行流程。
- 能识别本节应用上线前必须处理的余额、风险、延迟、失败和披露问题。

## 源码关联

- [crates/server/src/margin_metrics/poller.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/margin_metrics/poller.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/margin_metrics/metrics.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/margin_metrics/metrics.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/indexer/src/handlers/liquidation_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/liquidation_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/server/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/README.md)：REST API、数据库查询、健康检查或指标实现。

## 正文

风控系统要监控：RPC 可用性、Indexer lag、订单失败率、价格偏离、库存偏移、Margin 风险率、清算队列、数据库延迟。策略系统必须能在风控触发时停止发单。


风控系统要同时看市场风险、仓位风险和基础设施风险。Margin 清算、价格过期、checkpoint lag、机器人库存和 API 错误率都应进入同一套暂停与告警规则。

## 开发要点

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 检查问题

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
