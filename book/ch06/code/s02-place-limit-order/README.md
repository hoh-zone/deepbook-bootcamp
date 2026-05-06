# s02 限价单

目标：构造 `pool.place_limit_order` PTB，支持买单和卖单。

核心参数：

- `client_order_id`：本地生成，建议使用时间戳加递增序号。
- `order_type`：普通、IOC、FOK、post-only。
- `self_matching_option`：自成交策略。
- `price`：按 tick size 对齐后的整数。
- `quantity`：按 lot size 对齐后的 base 数量。
- `is_bid`：true 买，false 卖。
- `pay_with_deep`：当前版本优先 true。

运行方式：

```bash
pnpm install
pnpm tsx place-limit-order.ts --side bid --price 100000 --quantity 1000000
```

交易前检查：

- manager 内 quote/base 余额足够。
- manager 内 DEEP 余额足够支付手续费。
- `expire_timestamp` 晚于当前时间。
- post-only 不会穿越订单簿，否则 `EPOSTOrderCrossesOrderbook = 5`。
