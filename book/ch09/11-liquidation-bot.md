# ch09-11 清算机器人

[返回本章](README.md)

## 先看用户路径

这一节从 Margin 用户动作读起：围绕“清算机器人”，先把 manager、collateral、loan、trade 和 repay 的顺序排清楚，再看源码入口。

## 源码入口

重点阅读：

- [packages/margin_liquidation/sources/liquidation_vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/margin_liquidation/sources/liquidation_vault.move)
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [scripts/transactions/fundLiquidationVault.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/fundLiquidationVault.ts)
- `book/ch09/code/s05-liquidation-bot/README.md`

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 把 PTB 串起来

清算扫描器的职责是找出 `risk_ratio < liquidation_risk_ratio` 的 manager，并提交有利润的清算交易。

扫描流程：

1. 获取每个 Pool 的 base/quote MarginPool、risk params 和 Pyth price object。
2. 枚举注册的 `MarginManager` 或消费 indexer 事件。
3. 调读接口或 dev inspect 获取 `manager_state`、债务 shares 和风险率。
4. 过滤低于清算阈值的仓位。
5. 检查 liquidation vault 是否有足够债务资产。
6. 调 `liquidation_vault.liquidate_base` 或 `liquidate_quote`。

`fundLiquidationVault.ts` 展示了管理员注资：

```ts
client.deepbook.marginLiquidations.deposit(
  vaultId,
  liquidationAdminCapID[env],
  "USDC",
  50_000,
)(tx);
```

机器人需要监控 vault 的 base/quote/DEEP 余额。`liquidation_vault.move` 还提供 `swap_base_to_quote` 和 `swap_quote_to_base`，可用 DeepBook Pool 调整 vault 库存。

## 工程旁白

清算机器人不能只看缓存风险率。提交前应重新 dev inspect `liquidate_base` 或 `liquidate_quote`，确认仓位仍低于阈值、vault 有足够 debt asset、预期奖励覆盖 gas 和滑点。

vault 库存是机器人成功率的核心约束。base debt 需要 base 库存，quote debt 需要 quote 库存；`swap_base_to_quote` 和 `swap_quote_to_base` 可用于再平衡，但再平衡本身也依赖 DeepBook Pool 流动性和价格保护。

## Margin 应用判断

- 扫描器按 pool 分片，避免一个热池拖慢所有市场。
- 清算前记录 price timestamp 和 dev inspect 结果，便于解释失败或竞争丢单。
- 授权 trader 密钥只持有清算权限，不持有 admin 初始化权限。

## 动手检查

- 目标 manager 当前是否仍低于 liquidation threshold？
- vault 是否持有对应 debt asset，还是需要先 rebalance？
- 预期 user reward、pool reward、gas 和潜在 default 是否让这笔清算值得执行？
