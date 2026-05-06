# ch05-09 治理提案、投票与参数更新

[返回本章](README.md)

## 本节目标

- 读懂 治理提案、投票与参数更新 对 BalanceManager、Vault、Account 或费用参数的影响。
- 能画出本节相关 base、quote、DEEP 在 deposit、trade、settle、withdraw 中的资金方向。
- 能把权限、余额、版本和治理参数错误拆成可定位的 Move 模块。

## 源码关联

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move)：maker/taker fee、stake required、折扣等级和 epoch 参数读取。
- [packages/deepbook/sources/state/governance.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/governance.move)：治理提案、投票、参数更新、暂停和维护者能力边界。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 正文

`pool.submit_proposal` 接收 `taker_fee`、`maker_fee`、`stake_required`，要求 `BalanceManager` 有有效 proof，并由 `state.process_proposal` 检查账户 stake。`governance.add_proposal` 校验 fee 是否为 `FEE_MULTIPLE = 1000` 的倍数，并根据 stable 或 volatile 池限制上下界。whitelisted pool 不能改参数，错误码是 `EWhitelistedPoolCannotChange = 5`。

`pool.vote` 使用账户全部 voting power 投票。`governance.adjust_vote` 会把旧 proposal 的票移除，并给新 proposal 加票；当 proposal votes 超过 `quorum`，`next_trade_params` 更新为该 proposal。`governance.update` 在新 epoch 清空 proposals，把 `next_trade_params` 变成当前 `trade_params`，并发出 `TradeParamsUpdateEvent { taker_fee, maker_fee, stake_required }`。

投票权来自 `stake_to_voting_power`。100k DEEP 以下按 stake 线性计算，超过阈值后只增加平方根部分，避免单个巨鲸线性垄断。

补充阅读：治理提案、投票与参数更新 的资金主线是“钱包 Coin -> [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 托管余额 -> [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 记录成交后的 settled/owed -> [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 做实际资产进出”。撮合模块只产生成交结果，真正的资产划转要回到 BalanceManager 和 Vault 这两层看。

治理和维护能力影响的是参数生效、暂停和版本门禁，不应被前端当作普通余额错误处理。PTB dry run 返回这些 abort 时，提示语应引导用户等待版本升级或市场恢复，而不是要求用户继续充值。

## 开发要点

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 检查问题

- 治理提案、投票与参数更新 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
