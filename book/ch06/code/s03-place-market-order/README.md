# s03 市价单

目标：构造 `pool.place_market_order` PTB，并在提交前计算滑点。

源码行为：

- 买单使用 `constants::max_price()`。
- 卖单使用 `constants::min_price()`。
- order type 固定为 `immediate_or_cancel`。
- 未成交数量会取消，不进入订单簿。

运行方式：

```bash
pnpm install
pnpm tsx place-market-order.ts --side ask --quantity 1000000
```

Dry run 建议：

- 买入目标 base：调用 `get_quote_quantity_in`。
- 卖出目标 quote：调用 `get_base_quantity_in`。
- 给定输入数量：调用 `get_quantity_out`。

注意事项：

- `quantity` 仍然是 base 数量。
- 市价单不能 post-only，错误码 `EMarketOrderCannotBePostOnly = 7`。
- 前端应展示预估均价和最差成交价。
