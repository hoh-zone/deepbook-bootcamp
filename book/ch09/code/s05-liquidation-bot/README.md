# s05-liquidation-bot

目标：实现清算扫描器骨架，连接 `manager_state`、风险阈值、liquidation vault 余额和清算入口。

配置项：

- `NETWORK`
- `POOL_KEYS`
- `MARGIN_REGISTRY_ID`
- `LIQUIDATION_VAULT_ID`
- `BASE_PRICE_OBJECT_ID`
- `QUOTE_PRICE_OBJECT_ID`
- `TRADER_KEYPAIR`

扫描流程：

1. 读取每个 Pool 的 `PoolConfig`，得到 `liquidation_risk_ratio` 和 `target_liquidation_risk_ratio`。
2. 枚举注册的 `MarginManager`。
3. 对每个 manager 调读接口或 dev inspect，计算当前 risk ratio。
4. 过滤 `risk_ratio < liquidation_risk_ratio` 的仓位。
5. 判断 debt asset 是 base 还是 quote。
6. 检查 liquidation vault 对应资产余额。
7. 构造 `liquidate_base` 或 `liquidate_quote` 交易并 dry run。

链上入口：

- [packages/margin_liquidation/sources/liquidation_vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/margin_liquidation/sources/liquidation_vault.move)
- `liquidate_base<BaseAsset, QuoteAsset>`
- `liquidate_quote<BaseAsset, QuoteAsset>`

运行方式建议：

```bash
pnpm tsx index.ts --network mainnet --pool SUI_USDC --dry-run
```

生产注意事项：

- 只让已授权 trader 钱包提交清算交易。
- 每次清算前刷新 Pyth price object。
- 记录 abort code、gas、预估奖励和 vault 余额变化。
- vault 缺少 debt asset 时，不要盲目提交清算；先提示管理员注资或构造 vault swap。
