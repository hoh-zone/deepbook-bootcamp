# ch05-04 Vault 如何结算 balances_in 与 balances_out

[返回本章](README.md)

## 先沿资金问问题

先沿资金问问题。“Vault 如何结算 balances_in 与 balances_out”不是账面说明，而是在回答资产从钱包进入 BalanceManager、被订单锁定、成交后进入 settled/owed，最后由 Vault 收尾的哪一段。

## 源码入口

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：池子 `base_balance`/`quote_balance`、`balances_in`/`balances_out` 结算和资金安全断言。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 关键定义

Vault 是池子的真实资产保管层：

```move
public struct Vault<phantom BaseAsset, phantom QuoteAsset> has store {
    base_balance: Balance<BaseAsset>,
    quote_balance: Balance<QuoteAsset>,
    deep_balance: Balance<DEEP>,
}
```

这三个字段分别保存 base、quote 和 DEEP。撮合不会直接从 maker 钱包转给 taker 钱包，而是先计算 `balances_in` 和 `balances_out`，再由 Vault 与 `BalanceManager` 做差额结算。

结算时可以把逻辑理解成一个净额表：

```text
if balances_out.asset > balances_in.asset:
    Vault split 差额 -> deposit 到 BalanceManager

if balances_in.asset > balances_out.asset:
    BalanceManager withdraw 差额 -> join 到 Vault
```

这样写的好处是一次交易可以同时处理成交、费用、返还和剩余锁定释放，而不是在撮合循环中到处转 coin。

## 沿资金流看

`Vault<BaseAsset, QuoteAsset>` 持有 `base_balance`、`quote_balance`、`deep_balance`。它是池子的真实资产金库。`pool.place_order_int` 的资金路径是：

1. 生成 `OrderInfo`，调用 `book.create_order` 撮合或入簿。
2. `state.process_create` 更新 account，返回 `(settled, owed)`。
3. `vault.settle_balance_manager(settled, owed, balance_manager, trade_proof)` 执行资产划转。
4. `OrderInfo` 发出订单和成交事件。

在 `vault.settle_balance_manager` 中，如果 `balances_out.base() > balances_in.base()`，vault 从 `base_balance` split 差额并 deposit 到 manager；如果 `balances_in.base() > balances_out.base()`，manager 通过 `withdraw_with_proof` 扣出差额并 join 到 vault。quote 和 DEEP 同理。

`withdraw_settled_amounts_permissionless` 只能处理无 owed 的场景。`settle_balance_manager_permissionless` 要求 `balances_in` 三项都是 0，否则 `EHasOwedBalances = 8`；同时必须存在可领取余额，否则 `ENoBalanceToSettle = 7`。

> **资金旁白**：这条线不要从撮合引擎开始。钱包里的 `Coin<T>` 先进入 [balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 的托管余额，成交后的 settled/owed 记录在 [account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)，真正资产进出由 [vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 收尾。读费用和余额时，始终沿这条资金线核对。

余额类错误最常见于把钱包余额、manager 可用余额、open order 锁定量和 owed balance 混为一谈。交易前检查应分别展示这些数值，并说明哪一项会被本次 PTB 消耗。

## 资金安全判断

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 动手检查

- Vault 如何结算 balances_in 与 balances_out 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
