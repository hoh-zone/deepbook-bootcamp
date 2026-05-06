# ch14-07 套利机器人与闪电贷

[返回本章](README.md)

## 先定义产品场景

产品小节先想用户路径和失败路径。DeepBook 应用的难点往往不在单个 move call，而在状态同步、错误解释和风险降级。

## 源码入口

- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [crates/indexer/src/handlers/flash_loan_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/flash_loan_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。
- [crates/indexer/tests/snapshots/snapshot_tests__flash_loans__flashloans.snap](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/snapshots/snapshot_tests__flash_loans__flashloans.snap)：测试 fixture、断言或可复现验证材料。

## 把系统拼起来

套利机器人需要先计算完整路径利润，包括手续费、slippage、Gas 和失败概率。DeepBookV3 闪电贷适合在同一 PTB 内完成借出、交易、归还，但 hot potato 要求借出的资产必须在同一交易中等额归还。


套利机器人使用 DeepBook flash loan 时，借出、换路径、归还必须在同一 PTB 中完成。`flashloans` 只能用于统计闪电贷借出事件；它不是 Margin 债务，不能用来计算 `loan_borrowed` 或 `loan_repaid` 的风险敞口。

## 产品落地判断

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 动手检查

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
