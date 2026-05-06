# s04 治理查询

目标：查询池子的当前交易参数、下一 epoch 参数，以及治理事件。

查询对象：

- `pool.pool_trade_params()`：当前 `taker_fee`、`maker_fee`、`stake_required`。
- `pool.pool_next_trade_params()`：下一 epoch 参数。
- `pool.quorum()`：当前 epoch quorum。
- 事件 `TradeParamsUpdateEvent`：epoch 切换时的参数更新。

建议 API：

```bash
pnpm install
pnpm tsx governance-query.ts --pool 0x...
```

开发注意事项：

- proposal ID 在源码中使用 `BalanceManager` ID 作为 key。
- stable pool 和 volatile pool 的 fee 上下界不同。
- whitelisted pool 不能通过用户治理改费率，触发 `EWhitelistedPoolCannotChange = 5`。
