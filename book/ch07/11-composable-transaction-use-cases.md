# ch07-11 组合交易场景

[返回本章](README.md)

## 先看交易边界

读这一节时，把“组合交易场景”放进一个 PTB 里推演。重点不是能不能借出资产，而是 hot potato 如何强迫调用方在同一交易中归还。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：DeepBookV3 闪电贷唯一 Spot pool 入口，包含 `borrow_flashloan_base`、`borrow_flashloan_quote`、`return_flashloan_base`、`return_flashloan_quote`。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：底层 `FlashLoan` hot potato、`FlashLoanBorrowed` 事件、pool/asset/amount 校验和 Vault 余额 split/join。
- `book/ch07/code/s01-flashloan-move-wrapper`、`s02-flashloan-ptb`、`s03-flashloan-event-query`、`s04-flashloan-indexer-row`：本节对应的 wrapper、PTB、事件和表查询示例。

## 在一笔交易里走完

闪电贷适合在单笔 PTB 内完成可原子验证的策略：

- 跨池套利：从 A 池借 base，在 B 池换成 quote，再换回 base，归还本金，留下利润。
- 库存再平衡：临时借 quote 买回 base，使多个池子的做市库存回到目标比例。
- 批量清算辅助：借入临时资产完成外部协议操作，再用获得的资产归还。
- 迁移流动性：短暂补足某个资产缺口，完成多对象资金搬迁。

所有场景都必须在 PTB 内保证最终有精确本金数量的 `Coin<T>` 传给 return 函数。

> **安全边界**：本章的闪电贷只指 DeepBookV3 Spot pool 闪电贷，入口在 [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)，底层在 [vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)。不要把它和 `packages/deepbook_margin` 的普通借贷混淆；这里没有跨交易债务账户，也没有隔夜未还状态。

调用链是 `pool.move` borrow 入口取出 pool 内部 vault，`vault.move` 从 `base_balance` 或 `quote_balance` split 出 Coin，同时返回 `FlashLoan` hot potato。PTB 中间可以组合 swap、套利或清算式动作，但最后必须把同一个方向的本金和 `FlashLoan` 一起交回 return 入口。

## 组合交易提醒

- 借 base 必须用 `return_flashloan_base` 归还 base；借 quote 必须用 `return_flashloan_quote` 归还 quote。
- `FlashLoan` 不能跨交易保存，PTB 中必须让 borrow 返回值最终被 return 消费。
- 事件查询只把 `FlashLoanBorrowed` 当作成功交易的借出记录，不能推导出 Margin 债务状态。

## 动手检查

- 组合交易场景 的 `pool.move` 入口是哪一个，底层 `vault.move` 校验哪几个字段？
- 如果 PTB abort，是 pool id、资产方向、归还数量、hot potato 未消费，还是中间组合步骤失败？
- 这段逻辑是否仍明确区别 DeepBookV3 闪电贷和 `deepbook_margin` 普通借贷？
