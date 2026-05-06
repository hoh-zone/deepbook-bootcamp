# s04-liquidation-scenario

目标：构造一个清算场景，复刻 `margin_manager::liquidate` 中 debt repay、用户奖励、池奖励和坏账的推导。

建议输入：

```json
{
  "debt": 350,
  "assetsInDebtUnit": 400,
  "riskRatio": 1.05,
  "targetLiquidationRiskRatio": 1.25,
  "userLiquidationReward": 0.02,
  "poolLiquidationReward": 0.03,
  "repayCoinValue": 100
}
```

关键步骤：

1. 计算 `liquidation_reward_with_user_pool = 1 + user_reward + pool_reward`。
2. 用目标风险率计算理论 `debt_repay`。
3. 用 repay coin 和 pool reward 计算本次最多可还债务。
4. 把 repay amount 换算成 repay shares。
5. 输出 `debt_repaid`、`pool_reward`、`pool_default` 和清算者应得资产。

运行方式建议：

```bash
pnpm tsx index.ts fixtures/partial-liquidation.json
```

对照源码：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move) 的 `liquidate`。
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move) 的 `repay_liquidation`。
- [packages/margin_liquidation/sources/liquidation_vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/margin_liquidation/sources/liquidation_vault.move) 的 `liquidate_base` 和 `liquidate_quote`。
