# ch14-17 Margin 仪表盘 MVP

[返回本章](README.md)

## 本节目标

- 明确“Margin 仪表盘 MVP”在 DeepBook 应用闭环中的用户场景、交易构造和数据依赖。
- 能把 UI、SDK/PTB、Server API、Indexer 数据和监控信号连接成可运行流程。
- 能识别本节应用上线前必须处理的余额、风险、延迟、失败和披露问题。

## 源码关联

- [crates/indexer/src/handlers/loan_borrowed_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/loan_borrowed_handler.rs)：Margin 借款事件解析和 `loan_borrowed` 落库入口。
- [crates/indexer/src/handlers/loan_repaid_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/loan_repaid_handler.rs)：Margin 还款事件解析和 `loan_repaid` 落库入口。
- [crates/indexer/src/handlers/liquidation_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/liquidation_handler.rs)：清算事件解析和清算历史读模型。
- [packages/deepbook_margin/tests/margin_manager_tests.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/tests/margin_manager_tests.move)：MarginManager 行为与权限测试。

## 正文

MVP 范围：MarginManager、抵押品、债务、风险比率、借款、还款、清算历史。所有交易都应先 dry run 并显示风险变化。


Margin 仪表盘 MVP 要优先展示抵押、债务、风险率、清算价和历史借还。`loan_borrowed`、`loan_repaid` 是债务流水，`flashloans` 不是仓位数据，二者必须在 UI 和 API 命名上隔离。

## 开发要点

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 检查问题

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
