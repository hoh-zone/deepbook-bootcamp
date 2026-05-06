# s02-risk-ratio-calculator

目标：用离线数据复刻 `margin_manager.move` 的风险率直觉，辅助理解 `minBorrowRiskRatio`、`minWithdrawRiskRatio` 和 `liquidationRiskRatio`。

输入示例：

```json
{
  "base": "SUI",
  "quote": "USDC",
  "baseBalance": 90,
  "quoteBalance": 40,
  "debtAsset": "USDC",
  "debtAmount": 350,
  "basePriceInQuote": 4,
  "minWithdrawRiskRatio": 2,
  "minBorrowRiskRatio": 1.2499,
  "liquidationRiskRatio": 1.1
}
```

计算逻辑：

```text
assets_in_debt_unit = quoteBalance + baseBalance * basePriceInQuote
risk_ratio = assets_in_debt_unit / debtAmount
```

如果 debt asset 是 base，则反向换算：

```text
assets_in_debt_unit = baseBalance + quoteBalance / basePriceInQuote
```

运行方式建议：

```bash
pnpm tsx index.ts fixtures/sui-usdc-long.json
```

输出应包含：

- 当前风险率。
- 借款是否会触发 `EBorrowRiskRatioExceeded`。
- 提款是否会触发 `EWithdrawRiskRatioExceeded`。
- 是否满足 `registry.can_liquidate`。

注意：这个示例只用于教学。链上实现还会读取 DeepBook Pool 中未结算资产、open order 锁定余额、Pyth price object、stale price 和 confidence 检查。
