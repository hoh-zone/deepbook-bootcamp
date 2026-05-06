# s03-interest-accrual

目标：模拟 `margin_state.move` 与 `protocol_config.move` 的利息累计。

核心公式：

```text
utilization = total_borrow / total_supply
rate = base_rate + utilization * base_slope
```

当利用率超过 `optimal_utilization`：

```text
rate = base_rate + optimal_utilization * base_slope
     + (utilization - optimal_utilization) * excess_slope
```

然后按时间累计：

```text
interest = total_borrow * rate * elapsed_ms / year_ms
protocol_fees = interest * protocol_spread
new_total_supply = total_supply + interest - protocol_fees
new_total_borrow = total_borrow + interest
```

建议 fixture：

```json
{
  "totalSupply": 1000000,
  "totalBorrow": 800000,
  "elapsedMs": 86400000,
  "baseRate": 0,
  "baseSlope": 0.15,
  "optimalUtilization": 0.8,
  "excessSlope": 5,
  "protocolSpread": 0.2
}
```

运行方式建议：

```bash
pnpm tsx index.ts fixtures/usdc-80-utilization.json
```

检查输出：

- utilization。
- interest rate。
- interest。
- protocol fees。
- supply shares 对应金额变化。
- borrow shares 对应金额变化。

实现时保留小数输入，但输出要注明链上使用 9 位小数定点数，不能把 JavaScript 浮点结果直接用于生产交易构造。
