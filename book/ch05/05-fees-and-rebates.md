# ch05-05 maker fee、taker fee、protocol fee 与 rebate

[返回本章](README.md)

## 本节目标

- 读懂 maker fee、taker fee、protocol fee 与 rebate 对 BalanceManager、Vault、Account 或费用参数的影响。
- 能画出本节相关 base、quote、DEEP 在 deposit、trade、settle、withdraw 中的资金方向。
- 能把权限、余额、版本和治理参数错误拆成可定位的 Move 模块。

## 源码关联

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move)：maker/taker fee、stake required、折扣等级和 epoch 参数读取。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 正文

`trade_params.move` 只保存核心交易费率：`taker_fee`、`maker_fee`、`stake_required`。`OrderInfo` 记录成交过程中的 `paid_fees` 和 `maker_fees`，`OrderFilled` 事件会输出 `taker_fee`、`taker_fee_is_deep`、`maker_fee`、`maker_fee_is_deep`。

taker fee 是主动吃单的一方支付的费用。`pool.get_quantity_out` 和 `get_quantity_in` 会用 `governance.trade_params().taker_fee()` 做交易前估算。maker fee 是挂单成交时对应的 maker 费用或返佣参数，历史 maker fee 会进入 `state.history`，取消订单时 `process_cancel` 根据订单 epoch 的 maker fee 计算退款。

DeepBook 的手续费资产可以是 DEEP，也可以是输入资产。`pay_with_deep = true` 时，`deep_price.get_order_deep_price` 给出 `OrderDeepPrice`，`deep_price::fee_quantity` 把成交规模换算成 DEEP 数量；输入资产付费时，买单从 quote 侧加费，卖单从 base 侧加费，并应用 `constants::fee_penalty_multiplier()`。

本章把 protocol fee 理解为流入协议 vault 并后续可治理处理的费用。`pool.burn_deep` 会从历史累计的待销毁 DEEP 中调用 `vault.withdraw_deep_to_burn`，再用 `token::deep::burn` 销毁，并发出 `DeepBurned { pool_id, deep_burned }`。

补充阅读：maker fee、taker fee、protocol fee 与 rebate 的资金主线是“钱包 Coin -> [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 托管余额 -> [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 记录成交后的 settled/owed -> [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 做实际资产进出”。撮合模块只产生成交结果，真正的资产划转要回到 BalanceManager 和 Vault 这两层看。

费用阅读顺序建议从 [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move) 的当前参数开始，再看 [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 中 active stake 和 rebate 记录。`pay_with_deep` 会把手续费支付资产切到 DEEP，因此交易前检查必须同时覆盖 base/quote 余额、DEEP 余额和 deep price 是否可用。

## 开发要点

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 检查问题

- maker fee、taker fee、protocol fee 与 rebate 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
