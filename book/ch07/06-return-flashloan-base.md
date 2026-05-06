# ch07-06 return_flashloan_base 校验

[返回本章](README.md)

## 先看交易边界

这里先把问题放进同一笔交易里。“return_flashloan_base 校验”成立的前提是借出、使用、归还都在同一笔交易里完成；任何跨交易债务想象都应该立刻排除。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：DeepBookV3 闪电贷唯一 Spot pool 入口，包含 `borrow_flashloan_base`、`borrow_flashloan_quote`、`return_flashloan_base`、`return_flashloan_quote`。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：底层 `FlashLoan` hot potato、`FlashLoanBorrowed` 事件、pool/asset/amount 校验和 Vault 余额 split/join。
- `book/ch07/code/s01-flashloan-move-wrapper`、`s02-flashloan-ptb`、`s03-flashloan-event-query`、`s04-flashloan-indexer-row`：本节对应的 wrapper、PTB、事件和表查询示例。

## 在一笔交易里走完

`vault.return_flashloan_base` 的校验顺序：

1. `pool_id == flash_loan.pool_id`，否则 `EIncorrectLoanPool = 4`。
2. `type_name::with_defining_ids<BaseAsset>() == flash_loan.type_name`，否则 `EIncorrectTypeReturned = 5`。
3. `coin.value() == flash_loan.borrow_quantity`，否则 `EIncorrectQuantityReturned = 6`。
4. `self.base_balance.join(coin.into_balance<BaseAsset>())` 把资产归还 vault。
5. 解构 `FlashLoan`，消费 hot potato。

归还数量必须精确等于借出数量。多还或少还都失败。如果组合交易产生了利润，应先 split 出利润，只把本金数量传给 return 函数。

> **安全边界**：本章的闪电贷只指 DeepBookV3 Spot pool 闪电贷，入口在 [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)，底层在 [vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)。不要把它和 `packages/deepbook_margin` 的普通借贷混淆；这里没有跨交易债务账户，也没有隔夜未还状态。

归还校验关注三件事：pool id 是否匹配、借出资产方向是否匹配、归还数量是否等于 `FlashLoan` 记录的 amount。任意不一致都会让交易整体 abort，之前组合步骤的状态也不会落链。

## 组合交易提醒

- 借 base 必须用 `return_flashloan_base` 归还 base；借 quote 必须用 `return_flashloan_quote` 归还 quote。
- `FlashLoan` 不能跨交易保存，PTB 中必须让 borrow 返回值最终被 return 消费。
- 事件查询只把 `FlashLoanBorrowed` 当作成功交易的借出记录，不能推导出 Margin 债务状态。

## 动手检查

- return_flashloan_base 校验 的 `pool.move` 入口是哪一个，底层 `vault.move` 校验哪几个字段？
- 如果 PTB abort，是 pool id、资产方向、归还数量、hot potato 未消费，还是中间组合步骤失败？
- 这段逻辑是否仍明确区别 DeepBookV3 闪电贷和 `deepbook_margin` 普通借贷？
