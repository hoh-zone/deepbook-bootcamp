# ch07-09 FlashLoanBorrowed 事件与 flashloans 表

[返回本章](README.md)

## 本节目标

- 明确 FlashLoanBorrowed 事件与 flashloans 表 属于 DeepBookV3 Spot pool 闪电贷，入口固定在 `pool.move`，底层固定在 `vault/vault.move`。
- 能写出 base 或 quote 借出、组合使用、同交易归还的完整 PTB 顺序。
- 能区分 hot potato 闪电贷错误与 Margin 普通借贷、余额不足或交易组合失败。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：DeepBookV3 闪电贷唯一 Spot pool 入口，包含 `borrow_flashloan_base`、`borrow_flashloan_quote`、`return_flashloan_base`、`return_flashloan_quote`。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：底层 `FlashLoan` hot potato、`FlashLoanBorrowed` 事件、pool/asset/amount 校验和 Vault 余额 split/join。
- [crates/indexer/src/handlers/flash_loan_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/flash_loan_handler.rs)：把 `FlashLoanBorrowed` 事件写入 `flashloans` 表，注意它只记录借出事件。
- `book/ch07/code/s01-flashloan-move-wrapper`、`s02-flashloan-ptb`、`s03-flashloan-event-query`、`s04-flashloan-indexer-row`：本节对应的 wrapper、PTB、事件和表查询示例。

## 正文

`vault.borrow_flashloan_base` 和 `borrow_flashloan_quote` 都发出：

```move
public struct FlashLoanBorrowed has copy, drop {
    pool_id: ID,
    borrow_quantity: u64,
    type_name: TypeName,
}
```

Indexer 处理器在 `crates/indexer/src/handlers/flash_loan_handler.rs` 中把事件写入 `flashloans` 表，字段包括 `event_digest`、`digest`、`sender`、`checkpoint`、`checkpoint_timestamp_ms`、`package`、`pool_id`、`borrow_quantity`、`borrow = true`、`type_name`。

因为没有 return 事件，分析闪电贷成功与否应以包含该事件的交易最终成功为前提。失败交易不会产生最终可索引事件。

补充阅读：本章所有闪电贷都指 DeepBookV3 Spot pool 闪电贷，入口在 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)，底层在 [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)。不要把它和 `packages/deepbook_margin` 的普通借贷混淆；这里没有跨交易债务账户，也没有隔夜未还状态。

`FlashLoanBorrowed` 是借出方向事件，indexer 落库后只能证明成功交易里发生过 borrow。归还没有独立表行；如果交易最终失败，事件和表记录也不会成为成功状态的一部分。

## 开发要点

- 借 base 必须用 `return_flashloan_base` 归还 base；借 quote 必须用 `return_flashloan_quote` 归还 quote。
- `FlashLoan` 不能跨交易保存，PTB 中必须让 borrow 返回值最终被 return 消费。
- 事件查询只把 `FlashLoanBorrowed` 当作成功交易的借出记录，不能推导出 Margin 债务状态。

## 检查问题

- FlashLoanBorrowed 事件与 flashloans 表 的 `pool.move` 入口是哪一个，底层 `vault.move` 校验哪几个字段？
- 如果 PTB abort，是 pool id、资产方向、归还数量、hot potato 未消费，还是中间组合步骤失败？
- 这段逻辑是否仍明确区别 DeepBookV3 闪电贷和 `deepbook_margin` 普通借贷？
