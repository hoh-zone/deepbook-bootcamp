# ch05-03 存入、取出、锁定与结算

[返回本章](README.md)

## 本节目标

- 读懂 存入、取出、锁定与结算 对 BalanceManager、Vault、Account 或费用参数的影响。
- 能画出本节相关 base、quote、DEEP 在 deposit、trade、settle、withdraw 中的资金方向。
- 能把权限、余额、版本和治理参数错误拆成可定位的 Move 模块。

## 源码关联

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：池子 `base_balance`/`quote_balance`、`balances_in`/`balances_out` 结算和资金安全断言。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 正文

存入路径是 `balance_manager::deposit<T>(manager, coin, ctx)` 或 `deposit_with_cap<T>`。函数先发出 `BalanceEvent { balance_manager_id, asset, amount, deposit: true }`，再把 `Coin<T>` 转为 `Balance<T>`，通过 `deposit_with_proof` 合并到 `balances`。

取出路径是 `withdraw<T>`、`withdraw_all<T>` 或 `withdraw_with_cap<T>`。底层 `withdraw_with_proof<T>` 会按 `BalanceKey<T>` 取余额；如果余额小于提现量，抛出 `EBalanceManagerBalanceTooLow = 3`。提现成功后发出 `BalanceEvent { deposit: false }`。

下单时的锁定不是直接在 `BalanceManager` 内创建锁字段，而是在 `pool.place_order_int` 中经过 `state.process_create` 计算 `settled` 和 `owed`。`owed` 表示用户要付给 vault 的资产，`settled` 表示 vault 要付回用户的资产。对于挂在订单簿中的 maker 订单，订单对象记录了锁定数量；后续成交或取消时再把差额写入 account 的 settled balances。

补充阅读：存入、取出、锁定与结算 的资金主线是“钱包 Coin -> [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 托管余额 -> [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 记录成交后的 settled/owed -> [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 做实际资产进出”。撮合模块只产生成交结果，真正的资产划转要回到 BalanceManager 和 Vault 这两层看。

余额类错误最常见于把钱包余额、manager 可用余额、open order 锁定量和 owed balance 混为一谈。交易前检查应分别展示这些数值，并说明哪一项会被本次 PTB 消耗。

## 开发要点

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 检查问题

- 存入、取出、锁定与结算 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
