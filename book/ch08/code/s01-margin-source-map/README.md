# s01-margin-source-map

目标：生成 DeepBook Margin 模块依赖图，帮助读源码时确认控制流和资金流。

建议扫描路径：

- [packages/deepbook_margin/sources](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources)
- [packages/deepbook_margin/sources/helper](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper)
- [packages/deepbook_margin/sources/margin_pool](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool)
- [packages/margin_liquidation/sources/liquidation_vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/margin_liquidation/sources/liquidation_vault.move)

推荐输出节点：

- `margin_registry.move`：版本、PoolConfig、MarginPool ID、manager 注册和价格保护。
- `margin_manager.move`：账户、抵押、借款、还款、风险率、TPSL 和清算。
- `margin_pool.move`：供应、提款、借出、还款、费用和 rate limit。
- `pool_proxy.move`：下单、撤单、结算、reduce-only。
- `margin_state.move`：supply/borrow shares 与利息累计。
- `protocol_config.move`：利率曲线和池参数。
- `oracle.move`：Pyth 价格和 stale/confidence/EWMA 检查。
- `liquidation_vault.move`：授权清算资金库。

可以先写一个 TypeScript 脚本读取 Move 文件中的 `use deepbook_margin::...` 和 `use margin_liquidation::...`，生成 Mermaid：

```bash
pnpm tsx index.ts > margin-source-map.mmd
```

检查重点：

- `pool_proxy` 依赖 `margin_manager` 和 `margin_registry`，但不直接处理借贷池状态。
- `margin_manager` 调 `margin_pool.borrow` / `repay` / `repay_liquidation`，并调用 DeepBook Pool 结算和撤单。
- `liquidation_vault` 不计算核心清算公式，只提供资金、权限和资产回收。
