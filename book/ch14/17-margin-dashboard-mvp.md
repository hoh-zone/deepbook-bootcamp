# ch14-17 Margin 仪表盘 MVP

[返回本章](README.md)

## 先定义产品场景

Margin 仪表盘不是“借款历史列表”。它要回答用户最关心的四个问题：我抵押了什么，借了什么，当前风险率是多少，价格变化到哪里会触发清算。只要这四个问题有一个说不清，MVP 就还不能上线给真实用户。

## 源码入口

- [crates/indexer/src/handlers/loan_borrowed_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/loan_borrowed_handler.rs)：Margin 借款事件解析和 `loan_borrowed` 落库入口。
- [crates/indexer/src/handlers/loan_repaid_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/loan_repaid_handler.rs)：Margin 还款事件解析和 `loan_repaid` 落库入口。
- [crates/indexer/src/handlers/liquidation_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/liquidation_handler.rs)：清算事件解析和清算历史读模型。
- [packages/deepbook_margin/tests/margin_manager_tests.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/tests/margin_manager_tests.move)：MarginManager 行为与权限测试。

## 把系统拼起来

MVP 范围应包含：MarginManager、抵押品、债务、风险率、借款、还款、清算历史和 reduce-only 操作。所有写交易都应先 dry run，并在签名前展示交易后的风险变化。

Margin 仪表盘要优先展示抵押、债务、风险率、清算价和历史借还。`loan_borrowed`、`loan_repaid` 是债务流水，`flashloans` 是 Spot 闪电贷记录，不是仓位数据，二者必须在 UI 和 API 命名上隔离。

数据层至少需要两类刷新：链上直接读取 manager/pool 风险状态，用于交易前判断；Indexer/Server 读取历史借还和清算事件，用于时间线和报表。实时风控不能只依赖落库事件。

## 产品落地判断

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。
- 风险率展示要同时给出当前值、阈值、价格敏感性和数据更新时间。

## 动手检查

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
