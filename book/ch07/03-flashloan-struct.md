# ch07-03 FlashLoan 结构

[返回本章](README.md)

## 本节目标

- 明确 FlashLoan 结构 属于 DeepBookV3 Spot pool 闪电贷，入口固定在 `pool.move`，底层固定在 `vault/vault.move`。
- 能写出 base 或 quote 借出、组合使用、同交易归还的完整 PTB 顺序。
- 能区分 hot potato 闪电贷错误与 Margin 普通借贷、余额不足或交易组合失败。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：DeepBookV3 闪电贷唯一 Spot pool 入口，包含 `borrow_flashloan_base`、`borrow_flashloan_quote`、`return_flashloan_base`、`return_flashloan_quote`。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：底层 `FlashLoan` hot potato、`FlashLoanBorrowed` 事件、pool/asset/amount 校验和 Vault 余额 split/join。
- `book/ch07/code/s01-flashloan-move-wrapper`、`s02-flashloan-ptb`、`s03-flashloan-event-query`、`s04-flashloan-indexer-row`：本节对应的 wrapper、PTB、事件和表查询示例。

## 正文

`vault.move` 定义：

```move
public struct FlashLoan {
    pool_id: ID,
    borrow_quantity: u64,
    type_name: TypeName,
}
```

`pool_id` 绑定借出的池子；`borrow_quantity` 绑定必须归还的精确数量；`type_name` 绑定借出的资产类型。归还时三项都要匹配，否则 abort。

> **Move 技巧**：`FlashLoan` 是 hot potato 的典型用法。它没有 `drop`，也不会作为长期债务对象保存；调用者必须在同一笔交易里把它交给 return 函数消费掉，否则交易无法结束。

注意 `FlashLoan` 事件只有借出事件 `FlashLoanBorrowed`，没有单独的 return 事件。成功归还体现在同一交易没有 abort，vault 余额通过 coin join 回来。

补充阅读：本章所有闪电贷都指 DeepBookV3 Spot pool 闪电贷，入口在 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)，底层在 [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)。不要把它和 `packages/deepbook_margin` 的普通借贷混淆；这里没有跨交易债务账户，也没有隔夜未还状态。

归还校验关注三件事：pool id 是否匹配、借出资产方向是否匹配、归还数量是否等于 `FlashLoan` 记录的 amount。任意不一致都会让交易整体 abort，之前组合步骤的状态也不会落链。

## 开发要点

- 借 base 必须用 `return_flashloan_base` 归还 base；借 quote 必须用 `return_flashloan_quote` 归还 quote。
- `FlashLoan` 不能跨交易保存，PTB 中必须让 borrow 返回值最终被 return 消费。
- 事件查询只把 `FlashLoanBorrowed` 当作成功交易的借出记录，不能推导出 Margin 债务状态。

## 检查问题

- FlashLoan 结构 的 `pool.move` 入口是哪一个，底层 `vault.move` 校验哪几个字段？
- 如果 PTB abort，是 pool id、资产方向、归还数量、hot potato 未消费，还是中间组合步骤失败？
- 这段逻辑是否仍明确区别 DeepBookV3 闪电贷和 `deepbook_margin` 普通借贷？
