# ch13-08 Margin handlers

[返回本章](README.md)

## 先看数据问题

这里先从读模型需求开始。“Margin handlers”要回答的是链上事件如何变成可查询、可分页、可对账、可监控的读模型。

## 源码入口

- [crates/indexer/src/handlers/loan_borrowed_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/loan_borrowed_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/indexer/src/handlers/loan_repaid_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/loan_repaid_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/indexer/src/handlers/liquidation_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/liquidation_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/schema/migrations/2025-10-20-185028-0000_add_individual_margin_tables/up.sql](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/migrations/2025-10-20-185028-0000_add_individual_margin_tables/up.sql)：表结构、索引、Diesel schema 或迁移约束。
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)：Margin 合约对象、借贷、抵押、清算或风控逻辑。

## 关键定义

Margin 借款事件的 handler 和核心成交 handler 结构一致，但业务主键不同：这里围绕 `margin_manager_id`、`margin_pool_id`、loan amount 和 shares 建读模型。

```rust
define_handler! {
    name: LoanBorrowedHandler,
    processor_name: "loan_borrowed",
    event_type: LoanBorrowedEvent,
    db_model: LoanBorrowed,
    table: loan_borrowed,
    map_event: |event, meta| LoanBorrowed {
        event_digest: meta.event_digest(),
        digest: meta.digest(),
        sender: meta.sender(),
        checkpoint: meta.checkpoint(),
        checkpoint_timestamp_ms: meta.checkpoint_timestamp_ms(),
        package: meta.package(),
        margin_manager_id: event.margin_manager_id.to_string(),
        margin_pool_id: event.margin_pool_id.to_string(),
        loan_amount: event.loan_amount as i64,
        loan_shares: event.loan_shares as i64,
        onchain_timestamp: event.timestamp as i64,
    }
}
```

借款读模型必须同时保存 amount 和 shares。amount 解释这次事件的即时规模，shares 才能和后续利息累计后的债务余额对应起来。Margin 面板如果只展示历史 amount，会低估长期仓位的真实 debt。

## 落到读模型里

Margin 事件覆盖账户、借贷、清算、抵押、配置和 TPSL：

- `MarginManagerCreatedHandler`：Margin 账户创建。
- `LoanBorrowedHandler`、`LoanRepaidHandler`：普通借贷生命周期。
- `AssetSuppliedHandler`、`AssetWithdrawnHandler`：供应和退出 MarginPool。
- `DepositCollateralHandler`、`WithdrawCollateralHandler`：抵押品变化。
- `LiquidationHandler`：清算事件。
- `ConditionalOrderAddedHandler`、`ConditionalOrderExecutedHandler`、`ConditionalOrderCancelledHandler`：TPSL 条件订单。
- `InterestParamsUpdatedHandler`、`MarginPoolConfigUpdatedHandler`：利率和风险参数更新。

Margin 面板的关键查询是：当前债务、抵押品、历史借还、风险参数、清算历史。


Margin handlers 维护的是可审计的债务和抵押事件流。`loan_borrowed` 表示借款增加，`loan_repaid` 表示债务减少；它们和 `deposit_collateral`、`withdraw_collateral`、`liquidation` 一起才能解释仓位健康度，不能用 DeepBook `flashloans` 表替代。

## 数据系统判断

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 动手检查

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
