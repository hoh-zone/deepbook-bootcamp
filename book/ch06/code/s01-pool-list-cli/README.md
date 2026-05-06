# s01 池子列表 CLI

目标：列出可交易 DeepBookV3 pool，并展示 base/quote、tick size、lot size、min size 和费率。

数据来源：

- `PoolCreated` 事件。
- `registry.get_pool_id<BaseAsset, QuoteAsset>`。
- `pool.pool_trade_params()` 和 `pool.registered_pool()`。

运行方式：

```bash
pnpm install
pnpm tsx pool-list.ts --network testnet
```

输出字段建议：

```text
pool_id base_type quote_type tick_size lot_size min_size taker_fee maker_fee registered whitelisted
```

注意事项：

- base/quote 顺序不可反转。
- 缓存池列表时同时缓存 package version。
- whitelisted pool 可能 fee 为 0，stable pool 费率区间不同。
