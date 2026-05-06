# ch05-11 资金安全相关 abort code

[返回本章](README.md)

## 先沿资金问问题

这一节把“资金安全相关 abort code”放到余额流里看。只看钱包余额会误判交易可用性，真正要追的是权限 proof、托管余额、费用参数和结算路径。

## 源码入口

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：池子 `base_balance`/`quote_balance`、`balances_in`/`balances_out` 结算和资金安全断言。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 沿资金流看

重点错误码如下：

- `balance_manager.move`：`EInvalidOwner = 0`、`EInvalidTrader = 1`、`EInvalidProof = 2`、`EBalanceManagerBalanceTooLow = 3`、`EMaxCapsReached = 4`、`ECapNotInList = 5`。
- `vault.move`：`ENoBalanceToSettle = 7`、`EHasOwedBalances = 8` 与 permissionless settlement 相关。
- `pool.move`：`EInvalidQuantityIn = 6`、`EInvalidOrderBalanceManager = 9`、`EMinimumQuantityOutNotMet = 12`、`EInvalidStake = 13`、`EPoolNotRegistered = 14`、`EWrongPoolReferral = 20`。
- `governance.move`：`EInvalidMakerFee = 1`、`EInvalidTakerFee = 2`、`EProposalDoesNotExist = 3`、`EMaxProposalsReachedNotEnoughVotes = 4`、`EWhitelistedPoolCannotChange = 5`。

前端错误提示应把 Move abort 映射成可操作信息：余额不足提示充值到 BalanceManager；proof 错误提示重新选择 manager 或授权 cap；版本错误提示刷新池列表；治理 fee 错误提示检查 stable/volatile 上下界和 1000 倍数。

> **资金旁白**：这条线不要从撮合引擎开始。钱包里的 `Coin<T>` 先进入 [balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 的托管余额，成交后的 settled/owed 记录在 [account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)，真正资产进出由 [vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 收尾。读费用和余额时，始终沿这条资金线核对。

余额类错误最常见于把钱包余额、manager 可用余额、open order 锁定量和 owed balance 混为一谈。交易前检查应分别展示这些数值，并说明哪一项会被本次 PTB 消耗。

## 资金安全判断

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 动手检查

- 资金安全相关 abort code 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
