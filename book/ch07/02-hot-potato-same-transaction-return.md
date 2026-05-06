# ch07-02 hot potato 为什么强制同一交易归还

[返回本章](README.md)

## 本节目标

- 明确 hot potato 为什么强制同一交易归还 属于 DeepBookV3 Spot pool 闪电贷，入口固定在 `pool.move`，底层固定在 `vault/vault.move`。
- 能写出 base 或 quote 借出、组合使用、同交易归还的完整 PTB 顺序。
- 能区分 hot potato 闪电贷错误与 Margin 普通借贷、余额不足或交易组合失败。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：DeepBookV3 闪电贷唯一 Spot pool 入口，包含 `borrow_flashloan_base`、`borrow_flashloan_quote`、`return_flashloan_base`、`return_flashloan_quote`。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：底层 `FlashLoan` hot potato、`FlashLoanBorrowed` 事件、pool/asset/amount 校验和 Vault 余额 split/join。
- `book/ch07/code/s01-flashloan-move-wrapper`、`s02-flashloan-ptb`、`s03-flashloan-event-query`、`s04-flashloan-indexer-row`：本节对应的 wrapper、PTB、事件和表查询示例。

## 正文

`FlashLoan` 没有 `key`、`store`、`drop` 能力，只是一个必须被消费的普通值。借出函数返回 `(Coin<T>, FlashLoan)`；如果调用者不把 `FlashLoan` 传给对应 return 函数，交易结束时这个值无法被丢弃或存储，Move 类型系统会让交易失败。

这就是 hot potato 模式：债务凭证不能跨交易保存，只能在同一 PTB 内被还款函数解包。协议不需要记录借款账户、利息累计或清算状态。

补充阅读：本章所有闪电贷都指 DeepBookV3 Spot pool 闪电贷，入口在 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)，底层在 [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)。不要把它和 `packages/deepbook_margin` 的普通借贷混淆；这里没有跨交易债务账户，也没有隔夜未还状态。

归还校验关注三件事：pool id 是否匹配、借出资产方向是否匹配、归还数量是否等于 `FlashLoan` 记录的 amount。任意不一致都会让交易整体 abort，之前组合步骤的状态也不会落链。

## 开发要点

- 借 base 必须用 `return_flashloan_base` 归还 base；借 quote 必须用 `return_flashloan_quote` 归还 quote。
- `FlashLoan` 不能跨交易保存，PTB 中必须让 borrow 返回值最终被 return 消费。
- 事件查询只把 `FlashLoanBorrowed` 当作成功交易的借出记录，不能推导出 Margin 债务状态。

## 检查问题

- hot potato 为什么强制同一交易归还 的 `pool.move` 入口是哪一个，底层 `vault.move` 校验哪几个字段？
- 如果 PTB abort，是 pool id、资产方向、归还数量、hot potato 未消费，还是中间组合步骤失败？
- 这段逻辑是否仍明确区别 DeepBookV3 闪电贷和 `deepbook_margin` 普通借贷？
