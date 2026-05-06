# ch05-01 为什么使用 BalanceManager

[返回本章](README.md)

## 本节目标

- 读懂 为什么使用 BalanceManager 对 BalanceManager、Vault、Account 或费用参数的影响。
- 能画出本节相关 base、quote、DEEP 在 deposit、trade、settle、withdraw 中的资金方向。
- 能把权限、余额、版本和治理参数错误拆成可定位的 Move 模块。

## 源码关联

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 正文

DeepBook 不直接在撮合时从钱包对象扣款。链上订单可能跨交易存在，撮合发生时 maker 不一定参与当前交易，所以协议需要一个池子可验证的托管账户。`BalanceManager` 正是这个账户，它持有一组按资产类型索引的 `Balance<T>`，撮合时由 `TradeProof` 授权池子扣入或转出。

`balance_manager.move` 的 `BalanceManager` 包含 `owner`、`balances` 和 `allow_listed`。`balances` 是用户在 DeepBook 内的可用资产；`allow_listed` 保存被授权的 `TradeCap`、`DepositCap`、`WithdrawCap` ID。交易应用应把钱包余额和 `BalanceManager` 余额分开展示：钱包余额表示还未托管的 `Coin<T>`，`BalanceManager.balance<T>()` 才是可用于下单的余额。

开发时不要在提交订单前只检查钱包余额。`pool.place_limit_order` 和 `pool.place_market_order` 读取的是 `BalanceManager`，如果资产没有 deposit 到 manager，后续 `vault.settle_balance_manager` 会在 `withdraw_with_proof` 处因余额不足中止。

补充阅读：为什么使用 BalanceManager 的资金主线是“钱包 Coin -> [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 托管余额 -> [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 记录成交后的 settled/owed -> [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 做实际资产进出”。撮合模块只产生成交结果，真正的资产划转要回到 BalanceManager 和 Vault 这两层看。

余额类错误最常见于把钱包余额、manager 可用余额、open order 锁定量和 owed balance 混为一谈。交易前检查应分别展示这些数值，并说明哪一项会被本次 PTB 消耗。

## 开发要点

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 检查问题

- 为什么使用 BalanceManager 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
