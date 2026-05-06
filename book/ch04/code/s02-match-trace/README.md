# s02-match-trace

目标：输入一组 maker 订单和一个 taker 订单，输出 DeepBook 风格的撮合过程日志。

建议命令：

```bash
pnpm init
pnpm add -D tsx typescript
pnpm tsx trace.ts fixtures/three-levels.json
```

输入示例：

```json
{
  "asks": [
    { "price": "100", "quantity": "5" },
    { "price": "101", "quantity": "5" },
    { "price": "103", "quantity": "5" }
  ],
  "bids": [],
  "taker": { "side": "bid", "price": "101", "quantity": "12", "orderType": "no_restriction" }
}
```

期望日志：

```text
taker bid price=101 quantity=12
match ask price=100 base=5 quote=500 maker=completed
match ask price=101 base=5 quote=505 maker=completed
stop: next ask price=103 crosses taker limit
insert remaining bid quantity=2 price=101
```

源码对照：

- `OrderInfo.executed_quantity` 对应日志中的累计 base。
- `OrderInfo.cumulative_quote_quantity` 对应累计 quote。
- `Fill.completed` 决定 maker 是否从 Book 删除。
- `OrderInfo.order_inserted` 决定剩余 taker 是否成为 maker。

生产注意事项：真实 DeepBook 还会计算 fee、lot size、min size、过期订单和 `max_fills`，模拟器应逐步补齐这些规则。
