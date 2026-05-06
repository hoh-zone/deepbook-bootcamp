# ch14-04 下单表单和交易确认状态机

[返回本章](README.md)

## 先定义产品场景

产品小节先看上线闭环：交易、数据、风控、钱包和运维必须一起设计，不能只列功能按钮。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [scripts/transactions](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions)：TypeScript 交易构造或运行脚本样例。
- [crates/server/src/server.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/server.rs)：REST API、数据库查询、健康检查或指标实现。

## 把系统拼起来

交易状态机建议分为：编辑中、校验中、等待签名、已提交、链上确认、Indexer 确认、失败。不要只用钱包返回成功作为最终状态；交易确认和数据落库是两个阶段。


下单状态机至少包含编辑、模拟、钱包签名、提交、链上确认、Indexer 可见和失败恢复。每个状态都应记录交易 digest 或错误码，避免用户在 Indexer 延迟时重复提交同一意图。

## 产品落地判断

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 动手检查

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
