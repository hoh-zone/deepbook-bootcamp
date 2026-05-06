# ch05-08 deep_price 如何服务 DEEP 价格计算

[返回本章](README.md)

## 本节目标

- 读懂 deep_price 如何服务 DEEP 价格计算 对 BalanceManager、Vault、Account 或费用参数的影响。
- 能画出本节相关 base、quote、DEEP 在 deposit、trade、settle、withdraw 中的资金方向。
- 能把权限、余额、版本和治理参数错误拆成可定位的 Move 模块。

## 源码关联

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move)：maker/taker fee、stake required、折扣等级和 epoch 参数读取。
- [packages/deepbook/sources/vault/deep_price.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/deep_price.move)：为 DEEP 支付手续费和折扣计算提供换算价格。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 正文

`deep_price.move` 的 `DeepPrice` 分别维护 base 和 quote 的价格点以及累计值。`add_price_point` 要求同一方向价格点至少间隔一分钟，否则 `EDataPointRecentlyAdded = 1`；如果没有任何价格点，`calculate_order_deep_price` 抛出 `ENoDataPoints = 2`。

`pool.add_deep_price_point` 会用参考池和目标池更新 DEEP 价格，并发出 `PriceAdded { conversion_rate, timestamp, is_base_conversion, reference_pool, target_pool }`。交易时 `get_order_deep_price(whitelisted)` 会对 whitelisted pool 返回 `deep_per_asset = 0`，表示免手续费；普通池使用最近一天内价格点平均值。

应用下单前如果允许 `pay_with_deep = true`，必须先 dry run `get_quantity_out` 或 `get_quote_quantity_in`，确认需要的 DEEP 数量，并检查 manager 的 DEEP 余额加 settled DEEP 是否足够。

补充阅读：deep_price 如何服务 DEEP 价格计算 的资金主线是“钱包 Coin -> [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 托管余额 -> [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 记录成交后的 settled/owed -> [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 做实际资产进出”。撮合模块只产生成交结果，真正的资产划转要回到 BalanceManager 和 Vault 这两层看。

费用阅读顺序建议从 [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move) 的当前参数开始，再看 [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 中 active stake 和 rebate 记录。`pay_with_deep` 会把手续费支付资产切到 DEEP，因此交易前检查必须同时覆盖 base/quote 余额、DEEP 余额和 deep price 是否可用。

## 开发要点

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 检查问题

- deep_price 如何服务 DEEP 价格计算 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
