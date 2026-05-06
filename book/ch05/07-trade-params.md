# ch05-07 trade_params 中的交易参数

[返回本章](README.md)

## 先沿资金问问题

费用参数看起来只是几个数字，但它们会影响交易成本、DEEP staking 折扣、治理投票和前端报价。读这一节时不要只问 taker fee 是多少，要问这组参数在哪个 epoch 生效、对哪个账户生效、是否还能被治理改掉。

这也是应用很容易出错的地方：用户钱包有钱，不代表 BalanceManager 里有可用余额；用户 stake 足够，也不代表本次交易一定拿到折扣。

## 源码入口

按这个顺序读会比较稳：

- [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move)：先确认参数结构和折扣条件。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：再看账户 active stake、volume 和 rebate 如何影响费用。
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：最后回到真实可支付余额和 proof。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：看交易、stake、claim rebate 和治理入口如何把这些状态串起来。

## 关键定义

`TradeParams` 只保存三项参数，但它们会影响交易费、返佣展示、stake 门槛和治理投票权的解释。

```move
public struct TradeParams has copy, drop, store {
    taker_fee: u64,
    maker_fee: u64,
    stake_required: u64,
}

public(package) fun taker_fee_for_user(
    self: &TradeParams,
    active_stake: u64,
    volume_in_deep: u128,
): u64 {
    if (
        active_stake >= self.stake_required &&
        volume_in_deep >= (self.stake_required as u128)
    ) {
        self.taker_fee / 2
    } else {
        self.taker_fee
    }
}
```

这段折扣逻辑同时看 `active_stake` 和以 DEEP 计价的交易量。应用如果只展示“已 stake 足够”会误导用户；还必须检查最近成交量是否达到门槛，并且所有 fee 计算都要使用链上整数精度。

## 沿资金流看

`TradeParams` 的字段不多，但每个字段都跨了产品和协议两层：

- `taker_fee`：主动成交方费率，按 `constants::float_scaling()` 精度参与计算。
- `maker_fee`：被动挂单方费用或 rebate 相关参数。
- `stake_required`：获得 taker fee 折扣和参与治理的关键门槛。

`pool.pool_trade_params()` 返回当前参数，`pool.pool_next_trade_params()` 返回下一 epoch 参数。这个区分对前端很实际：如果治理投票刚通过，交易预览应该告诉用户“当前交易仍按当前参数计算”，不要提前展示下一 epoch 的费率。

> **资金旁白**：这条线不要从撮合引擎开始。钱包里的 `Coin<T>` 先进入 [balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 的托管余额，成交后的 settled/owed 记录在 [account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)，真正资产进出由 [vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 收尾。读费用和余额时，始终沿这条资金线核对。

费用阅读顺序建议从 [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move) 的当前参数开始，再看 [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 中 active stake 和 rebate 记录。`pay_with_deep` 会把手续费支付资产切到 DEEP，因此交易前检查必须同时覆盖 base/quote 余额、DEEP 余额和 deep price 是否可用。

## 资金安全判断

- 交易前同时检查 BalanceManager 余额、proof、pool version、当前 fee 参数和 DEEP 余额。
- 计算折扣时必须同时看 active stake 和 DEEP 计价 volume；只满足一项不会减半。
- 历史成交展示要保存当时的 epoch 参数，否则事后用当前参数重算会对不上链上事件。

## 动手检查

- 一个用户 stake 已达标但 volume 未达标，`taker_fee_for_user` 会返回什么？
- 治理更新了 next params 后，当前 epoch 的交易预览应该显示哪组参数？
- 如果用户选择用 DEEP 支付手续费，交易前要多检查哪一种余额？
