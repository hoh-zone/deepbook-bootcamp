# ch14-15 专业交易终端 MVP

[返回本章](README.md)

## 先定义产品场景

产品小节先想用户路径和失败路径。DeepBook 应用的难点往往不在单个 move call，而在状态同步、错误解释和风险降级。

## 源码入口

- [crates/server/src/reader.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/reader.rs)：REST API、数据库查询、健康检查或指标实现。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [crates/indexer/src/handlers/order_update_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_update_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/indexer/src/handlers/order_fill_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_fill_handler.rs)：事件解析、字段映射或业务流水落库入口。

## 把系统拼起来

第一版的边界要收紧：池子列表、订单簿、最近成交、BalanceManager、限价单、市价单、撤单、订单历史。先做交易闭环，再做高级图表。


专业交易终端 MVP 至少应完成池子选择、订单簿、下单、交易确认、历史成交和错误提示。MVP 可以牺牲高级图表，但不能缺少 digest 跟踪、Indexer 延迟提示和余额校验。

## 产品落地判断

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 动手检查

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
