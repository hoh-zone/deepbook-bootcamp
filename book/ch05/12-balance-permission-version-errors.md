# ch05-12 余额不一致、权限错误、版本错误

[返回本章](README.md)

## 本节目标

- 读懂 余额不一致、权限错误、版本错误 对 BalanceManager、Vault、Account 或费用参数的影响。
- 能画出本节相关 base、quote、DEEP 在 deposit、trade、settle、withdraw 中的资金方向。
- 能把权限、余额、版本和治理参数错误拆成可定位的 Move 模块。

## 源码关联

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：池子 `base_balance`/`quote_balance`、`balances_in`/`balances_out` 结算和资金安全断言。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 正文

余额不一致通常来自三类问题。第一，用户只看钱包余额，没有把资产 deposit 到 `BalanceManager`。第二，用户有 `settled_balances` 但没有调用 `withdraw_settled_amounts` 把它们落回 manager。第三，使用输入资产付费时没有额外预留 fee，导致下单模拟成功数量和实际可扣数量不同。

权限错误多发生在托管机器人。`TradeCap` 只能生成交易 proof，不能提现；`WithdrawCap` 可以从 manager 提现，应严格隔离。撤销 cap 后，继续用旧 cap 会在 allow list 校验失败。

版本错误来自 registry 或 pool allowed versions。应用缓存池列表时要带版本字段，提交 PTB 前做一次轻量刷新或 dry run。

补充阅读：余额不一致、权限错误、版本错误 的资金主线是“钱包 Coin -> [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 托管余额 -> [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 记录成交后的 settled/owed -> [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 做实际资产进出”。撮合模块只产生成交结果，真正的资产划转要回到 BalanceManager 和 Vault 这两层看。

权限路径要区分 owner、`TradeCap`、`WithdrawCap` 和 `DepositCap`。应用给机器人做授权时通常只需要交易能力，不应同时授予提款能力；否则撮合之外的资金提取风险会扩大。

## 开发要点

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 检查问题

- 余额不一致、权限错误、版本错误 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
