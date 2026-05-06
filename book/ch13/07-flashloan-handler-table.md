# ch13-07 Flash Loan handler 与 `flashloans` 表

[返回本章](README.md)

## 本节目标

- 明确 DeepBookV3 `FlashLoanBorrowed` 如何进入 `flashloans` 表，以及它和 Margin 借贷表的差异。
- 能定位本节涉及的 handler、schema、Server 模块和部署入口，并说明它们之间的数据流。
- 能把“Flash Loan handler 与 `flashloans` 表”用于交易终端、行情、Margin 面板或机器人风控的真实查询设计。

## 源码关联

- [crates/indexer/src/handlers/flash_loan_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/flash_loan_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [crates/indexer/tests/snapshots/snapshot_tests__flash_loans__flashloans.snap](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/snapshots/snapshot_tests__flash_loans__flashloans.snap)：测试 fixture、断言或可复现验证材料。

## 正文

DeepBookV3 闪电贷事件来自 `packages/deepbook/sources/vault/vault.move` 的 `FlashLoanBorrowed`。Indexer 中的 `FlashLoanHandler` 把它写入 `flashloans` 表：

- `pool_id`：借出资产所在池子。
- `borrow_quantity`：借出数量。
- `type_name`：借出的资产类型。
- `borrow`：当前 handler 固定写为 `true`，表示借出事件。

注意：链上 `FlashLoan` hot potato 会强制同一交易归还资产，因此表里记录的是借出事件，不代表存在跨交易债务。不要把它和 Margin 的 `loan_borrowed`、`loan_repaid` 混为一类。


需要特别区分两类借贷语义：`flashloans` 记录 DeepBook vault 的 `FlashLoanBorrowed` 借出事件，hot potato 保证同一 PTB 内归还，因此它不是跨交易负债；Margin 的 `loan_borrowed`、`loan_repaid` 记录用户在 MarginPool 中形成和偿还的债务生命周期，风险率、利息和清算都应读取 Margin 表。

## 开发要点

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 检查问题

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
