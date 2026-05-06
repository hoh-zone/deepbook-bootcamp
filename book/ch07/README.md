# ch07 DeepBookV3 闪电贷与组合交易

## 本章目标

本章只讨论 DeepBookV3 Spot pool 的闪电贷。DeepBookV3 确实存在闪电贷，入口在 `packages/deepbook/sources/pool.move`，底层实现位于 `packages/deepbook/sources/vault/vault.move`。它使用 hot potato 模式强制同一交易归还资产，没有跨交易债务状态，不要和 `packages/deepbook_margin` 的普通借贷混淆。

## 本章学习阶梯

- L2 先理解“同一交易借出再归还”的资源闭环。
- L3 读 `FlashLoan` hot potato、borrow/return 和 Vault 校验。
- L4 构造一个成功和一个失败的闪电贷 PTB。
- L5 能把闪电贷放进套利、清算或组合交易，并设计风险检查。

## 关键定义卡片

DeepBookV3 闪电贷的定义在 `vault.move`，入口暴露在 `pool.move`：

```move
public struct FlashLoan {
    pool_id: ID,
    borrow_quantity: u64,
    type_name: TypeName,
}
```

`FlashLoan` 没有 `drop`，所以它是 hot potato。借出后必须在同一笔交易里归还并消费掉。三个字段就是归还校验条件：池子必须相同，资产类型必须相同，数量必须完全相同。

```move
public fun borrow_flashloan_base<BaseAsset, QuoteAsset>(
    self: &mut Pool<BaseAsset, QuoteAsset>,
    base_amount: u64,
    ctx: &mut TxContext,
): (Coin<BaseAsset>, FlashLoan)

public fun return_flashloan_base<BaseAsset, QuoteAsset>(
    self: &mut Pool<BaseAsset, QuoteAsset>,
    coin: Coin<BaseAsset>,
    flash_loan: FlashLoan,
)
```

借 quote 也有对应的 quote 版本。不要把这里和 Margin 借款混淆：闪电贷没有跨交易债务 shares，也没有还款计划，失败就整笔交易 abort。

## 源码地图

- `packages/deepbook/sources/pool.move`：`borrow_flashloan_base`、`borrow_flashloan_quote`、`return_flashloan_base`、`return_flashloan_quote`。
- `packages/deepbook/sources/vault/vault.move`：`FlashLoan`、`FlashLoanBorrowed`、底层借出和归还校验。
- `crates/indexer/src/handlers/flash_loan_handler.rs`：把 `FlashLoanBorrowed` 事件写入 `flashloans` 表。

## 小节目录

- [01 DeepBookV3 闪电贷入口](01-flashloan-entrypoints.md)
- [02 hot potato 为什么强制同一交易归还](02-hot-potato-same-transaction-return.md)
- [03 FlashLoan 结构](03-flashloan-struct.md)
- [04 borrow_flashloan_base 路径](04-borrow-flashloan-base.md)
- [05 borrow_flashloan_quote 路径](05-borrow-flashloan-quote.md)
- [06 return_flashloan_base 校验](06-return-flashloan-base.md)
- [07 return_flashloan_quote 校验](07-return-flashloan-quote.md)
- [08 错误码](08-flashloan-error-codes.md)
- [09 FlashLoanBorrowed 事件与 flashloans 表](09-flashloan-borrowed-event-table.md)
- [10 与 Margin 借贷的差异](10-flashloan-vs-margin-loans.md)
- [11 组合交易场景](11-composable-transaction-use-cases.md)
- [12 没有跨交易债务状态的安全边界](12-no-cross-transaction-debt.md)
- [13 借出再归还 PTB](13-borrow-and-return-ptb.md)
- [14 失败闪电贷练习](14-failing-flashloan-exercise.md)

## 本章代码

- `book/ch07/code/s01-flashloan-move-wrapper/`：Move wrapper 调用借出和归还。
- `book/ch07/code/s02-flashloan-ptb/`：TypeScript PTB 构造示例。
- `book/ch07/code/s03-flashloan-event-query/`：查询 `FlashLoanBorrowed` 事件。
- `book/ch07/code/s04-flashloan-indexer-row/`：展示 `flashloans` 表记录。

## Move 高阶穿插点

- 闪电贷是 hot potato 模式的高级案例：资源必须在同一交易里被消费，类型系统参与安全保证。
- 借 base 和借 quote 要分别追踪 type name、pool id 和数量，任何一个维度错都会 abort。
- 组合交易读法是先保证资源闭环，再谈套利、清算或跨协议收益。

## 常见错误

- 把 DeepBookV3 闪电贷写成 Margin 借贷。
- 以为可以跨交易归还。
- 多还或少还本金。
- 借 base 后用 quote return。
- 只查询 borrow 事件，不检查交易是否成功。

## 本章检查清单

- 能指出闪电贷入口在 `pool.move`，底层在 `vault/vault.move`。
- 能解释 `FlashLoan` hot potato 的三个字段。
- 能写出 base 和 quote 借还路径。
- 能列出六个闪电贷错误码。
- 能说明 indexer `flashloans` 表只有 borrow 方向事件。

## 进阶练习

1. 编写一个 PTB，借 base 后调用一次 swap，再归还 base。
2. 构造错误 pool 归还的 dry run，记录 abort code。
3. 写 SQL 统计每个 pool 每日闪电贷借出总量和最大单笔数量。
