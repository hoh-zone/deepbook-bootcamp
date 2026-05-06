# ch14-08 Margin 应用

[返回本章](README.md)

## 本节目标

- 明确“Margin 应用”在 DeepBook 应用闭环中的用户场景、交易构造和数据依赖。
- 能把 UI、SDK/PTB、Server API、Indexer 数据和监控信号连接成可运行流程。
- 能识别本节应用上线前必须处理的余额、风险、延迟、失败和披露问题。

## 源码关联

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)：Margin 合约对象、借贷、抵押、清算或风控逻辑。
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)：Margin 合约对象、借贷、抵押、清算或风控逻辑。
- [crates/indexer/src/handlers/loan_borrowed_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/loan_borrowed_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/indexer/src/handlers/loan_repaid_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/loan_repaid_handler.rs)：事件解析、字段映射或业务流水落库入口。

## 正文

Margin 应用的核心不是下单，而是风险展示。用户必须看到抵押品、债务、风险比率、可借额度、利息和清算阈值。每次借款、交易、还款前都应 dry run。


Margin 应用要把抵押、借款、偿还、清算价和风险参数放在同一视图。用户看到的健康度应来自 MarginPool 参数、oracle 价格和债务事件流，不能只看钱包余额或最近成交价。

## 开发要点

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 检查问题

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
