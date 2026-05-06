# ch07-04 borrow_flashloan_base 路径

[返回本章](README.md)

## 本节目标

- 明确 borrow_flashloan_base 路径 属于 DeepBookV3 Spot pool 闪电贷，入口固定在 `pool.move`，底层固定在 `vault/vault.move`。
- 能写出 base 或 quote 借出、组合使用、同交易归还的完整 PTB 顺序。
- 能区分 hot potato 闪电贷错误与 Margin 普通借贷、余额不足或交易组合失败。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：DeepBookV3 闪电贷唯一 Spot pool 入口，包含 `borrow_flashloan_base`、`borrow_flashloan_quote`、`return_flashloan_base`、`return_flashloan_quote`。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：底层 `FlashLoan` hot potato、`FlashLoanBorrowed` 事件、pool/asset/amount 校验和 Vault 余额 split/join。
- `book/ch07/code/s01-flashloan-move-wrapper`、`s02-flashloan-ptb`、`s03-flashloan-event-query`、`s04-flashloan-indexer-row`：本节对应的 wrapper、PTB、事件和表查询示例。

## 源码定义

Pool 层公开入口：

```move
public fun borrow_flashloan_base<BaseAsset, QuoteAsset>(
    self: &mut Pool<BaseAsset, QuoteAsset>,
    base_amount: u64,
    ctx: &mut TxContext,
): (Coin<BaseAsset>, FlashLoan)
```

Vault 层底层实现返回两个资源：

```move
public(package) fun borrow_flashloan_base<BaseAsset, QuoteAsset>(
    self: &mut Vault<BaseAsset, QuoteAsset>,
    pool_id: ID,
    borrow_quantity: u64,
    ctx: &mut TxContext,
): (Coin<BaseAsset>, FlashLoan)
```

返回值 `(Coin<BaseAsset>, FlashLoan)` 必须一起读。`Coin<BaseAsset>` 是你借到的资产，`FlashLoan` 是必须在同一 PTB 中归还时消费的证明。没有 `FlashLoan`，你无法完成 return；没有 return，交易无法结束。

## 正文

调用路径：

1. 应用在 PTB 中调用 `pool::borrow_flashloan_base<BaseAsset, QuoteAsset>(&mut pool, base_amount, ctx)`。
2. pool 加载 `PoolInner`，调用 `vault.borrow_flashloan_base(self.pool_id, base_amount, ctx)`。
3. vault 校验 `borrow_quantity > 0`，否则 `EInvalidLoanQuantity = 3`。
4. vault 校验 `base_balance.value() >= borrow_quantity`，否则 `ENotEnoughBaseForLoan = 1`。
5. vault 从 `base_balance` split 指定数量，转成 `Coin<BaseAsset>`。
6. vault 构造 `FlashLoan { pool_id, borrow_quantity, type_name: BaseAsset }`。
7. 发出 `FlashLoanBorrowed { pool_id, borrow_quantity, type_name }`。

拿到的 `Coin<BaseAsset>` 可以继续在同一 PTB 中用于 swap、套利、再平衡或其它组合操作。

补充阅读：本章所有闪电贷都指 DeepBookV3 Spot pool 闪电贷，入口在 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)，底层在 [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)。不要把它和 `packages/deepbook_margin` 的普通借贷混淆；这里没有跨交易债务账户，也没有隔夜未还状态。

调用链是 `pool.move` borrow 入口取出 pool 内部 vault，`vault.move` 从 `base_balance` 或 `quote_balance` split 出 Coin，同时返回 `FlashLoan` hot potato。PTB 中间可以组合 swap、套利或清算式动作，但最后必须把同一个方向的本金和 `FlashLoan` 一起交回 return 入口。

## 开发要点

- 借 base 必须用 `return_flashloan_base` 归还 base；借 quote 必须用 `return_flashloan_quote` 归还 quote。
- `FlashLoan` 不能跨交易保存，PTB 中必须让 borrow 返回值最终被 return 消费。
- 事件查询只把 `FlashLoanBorrowed` 当作成功交易的借出记录，不能推导出 Margin 债务状态。

## 检查问题

- borrow_flashloan_base 路径 的 `pool.move` 入口是哪一个，底层 `vault.move` 校验哪几个字段？
- 如果 PTB abort，是 pool id、资产方向、归还数量、hot potato 未消费，还是中间组合步骤失败？
- 这段逻辑是否仍明确区别 DeepBookV3 闪电贷和 `deepbook_margin` 普通借贷？
