# ch14-15 专业交易终端 MVP

[返回本章](README.md)

## 本节目标

- 明确“交易终端 MVP”在 DeepBook 应用闭环中的用户场景、交易构造和数据依赖。
- 能把 UI、SDK/PTB、Server API、Indexer 数据和监控信号连接成可运行流程。
- 能识别本节应用上线前必须处理的余额、风险、延迟、失败和披露问题。

## 源码关联

- [crates/server/src/reader.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/reader.rs)：REST API、数据库查询、健康检查或指标实现。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [crates/indexer/src/handlers/order_update_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_update_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/indexer/src/handlers/order_fill_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_fill_handler.rs)：事件解析、字段映射或业务流水落库入口。

## 正文

MVP 范围：池子列表、订单簿、最近成交、BalanceManager、限价单、市价单、撤单、订单历史。先做交易闭环，再做高级图表。


专业交易终端 MVP 至少应完成池子选择、订单簿、下单、交易确认、历史成交和错误提示。MVP 可以牺牲高级图表，但不能缺少 digest 跟踪、Indexer 延迟提示和余额校验。

## 开发要点

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 检查问题

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
