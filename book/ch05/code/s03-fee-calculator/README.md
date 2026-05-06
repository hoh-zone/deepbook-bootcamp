# s03 费用估算器

目标：根据池子参数、订单方向和付费资产估算 taker fee、maker fee 与 DEEP 需求。

输入：

- `pool.pool_trade_params()` 返回的 `taker_fee`、`maker_fee`、`stake_required`。
- `pool.get_quantity_out` 或 `get_quantity_out_input_fee` 的 dry run 结果。
- 用户账户的 `active_stake`、`taker_volume`、`maker_volume`。

估算规则：

- `pay_with_deep = true`：使用 `deep_price::OrderDeepPrice` 把 base 或 quote 成交规模换算成 DEEP，再乘以 taker fee。
- `pay_with_deep = false`：买单从 quote 侧扣输入资产 fee，卖单从 base 侧扣输入资产 fee。
- 用户同时满足 `active_stake >= stake_required` 和 `volume_in_deep >= stake_required` 时，`taker_fee_for_user` 返回半价 taker fee。

运行方式：

```bash
pnpm install
pnpm tsx fee-calculator.ts --side bid --quantity 100000000 --pay-with-deep true
```

错误处理：

- 没有 DEEP 价格点时，`deep_price` 可能抛出 `ENoDataPoints = 2`。
- 输入资产付费需要预留 penalty multiplier 后的额外数量。
- whitelisted pool 的 DEEP fee 价格会返回 0，不要误报缺少 DEEP。
