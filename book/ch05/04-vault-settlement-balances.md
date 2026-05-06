# ch05-04 Vault 如何结算 balances_in 与 balances_out

[返回本章](README.md)

## 本节目标

- 读懂 Vault 如何结算 balances_in 与 balances_out 对 BalanceManager、Vault、Account 或费用参数的影响。
- 能画出本节相关 base、quote、DEEP 在 deposit、trade、settle、withdraw 中的资金方向。
- 能把权限、余额、版本和治理参数错误拆成可定位的 Move 模块。

## 源码关联

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：池子 `base_balance`/`quote_balance`、`balances_in`/`balances_out` 结算和资金安全断言。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 正文

`Vault<BaseAsset, QuoteAsset>` 持有 `base_balance`、`quote_balance`、`deep_balance`。它是池子的真实资产金库。`pool.place_order_int` 的资金路径是：

1. 生成 `OrderInfo`，调用 `book.create_order` 撮合或入簿。
2. `state.process_create` 更新 account，返回 `(settled, owed)`。
3. `vault.settle_balance_manager(settled, owed, balance_manager, trade_proof)` 执行资产划转。
4. `OrderInfo` 发出订单和成交事件。

在 `vault.settle_balance_manager` 中，如果 `balances_out.base() > balances_in.base()`，vault 从 `base_balance` split 差额并 deposit 到 manager；如果 `balances_in.base() > balances_out.base()`，manager 通过 `withdraw_with_proof` 扣出差额并 join 到 vault。quote 和 DEEP 同理。

`withdraw_settled_amounts_permissionless` 只能处理无 owed 的场景。`settle_balance_manager_permissionless` 要求 `balances_in` 三项都是 0，否则 `EHasOwedBalances = 8`；同时必须存在可领取余额，否则 `ENoBalanceToSettle = 7`。

补充阅读：Vault 如何结算 balances_in 与 balances_out 的资金主线是“钱包 Coin -> [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 托管余额 -> [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 记录成交后的 settled/owed -> [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 做实际资产进出”。撮合模块只产生成交结果，真正的资产划转要回到 BalanceManager 和 Vault 这两层看。

余额类错误最常见于把钱包余额、manager 可用余额、open order 锁定量和 owed balance 混为一谈。交易前检查应分别展示这些数值，并说明哪一项会被本次 PTB 消耗。

## 开发要点

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 检查问题

- Vault 如何结算 balances_in 与 balances_out 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
