# ch05-06 DEEP staking 如何影响费用等级

[返回本章](README.md)

## 本节目标

- 读懂 DEEP staking 如何影响费用等级 对 BalanceManager、Vault、Account 或费用参数的影响。
- 能画出本节相关 base、quote、DEEP 在 deposit、trade、settle、withdraw 中的资金方向。
- 能把权限、余额、版本和治理参数错误拆成可定位的 Move 模块。

## 源码关联

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move)：maker/taker fee、stake required、折扣等级和 epoch 参数读取。
- [packages/deepbook/sources/vault/deep_price.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/deep_price.move)：为 DEEP 支付手续费和折扣计算提供换算价格。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 正文

`pool.stake` 要求 `amount > 0`，否则 `EInvalidStake = 13`。它调用 `state.process_stake`，账户侧 `account.add_stake` 会把 stake 加入 `inactive_stake`，并把同等 DEEP 加入 `owed_balances`，随后 vault 从 `BalanceManager` 扣 DEEP。

`account.update` 在新 epoch 把 `inactive_stake` 转入 `active_stake`。`trade_params::taker_fee_for_user(active_stake, volume_in_deep)` 规定：当用户 active stake 大于等于 `stake_required`，且按 DEEP 计价的交易量也大于等于 `stake_required`，taker fee 减半。否则使用完整 taker fee。

`pool.unstake` 会调用 `state.process_unstake`，账户的 `remove_stake` 把 active 和 inactive stake 清零，并把 DEEP 加回 `settled_balances`。应用侧应提示 unstake 会清除投票状态，并且 staking 折扣不是当前交易立即生效，而依赖 epoch 刷新和历史交易量。

补充阅读：DEEP staking 如何影响费用等级 的资金主线是“钱包 Coin -> [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 托管余额 -> [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 记录成交后的 settled/owed -> [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 做实际资产进出”。撮合模块只产生成交结果，真正的资产划转要回到 BalanceManager 和 Vault 这两层看。

费用阅读顺序建议从 [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move) 的当前参数开始，再看 [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 中 active stake 和 rebate 记录。`pay_with_deep` 会把手续费支付资产切到 DEEP，因此交易前检查必须同时覆盖 base/quote 余额、DEEP 余额和 deep price 是否可用。

## 开发要点

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 检查问题

- DEEP staking 如何影响费用等级 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
