# s01 Predict source map

目标：从 [packages/predict/sources](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources) 生成模块依赖图，帮助阅读 ch10。

建议命令：

```bash
cd deepbookv3
rg -n "^module |^use " packages/predict/sources
```

可扩展脚本思路：

1. 扫描 `packages/predict/sources/**/*.move`。
2. 提取 `module deepbook_predict::name;` 作为节点。
3. 提取 `use deepbook_predict::{...}` 和 `use deepbook_predict::module` 作为内部边。
4. 输出 Mermaid graph 或 DOT。

阅读重点：

- `predict.move` 依赖 vault、oracle、oracle_config、pricing_config、risk_config、fee_reserve、predict_manager 和 range_key。
- `registry.move` 是 admin 与对象创建入口。
- `vault.move` 依赖 `strike_matrix.move` 和 `range_key.move`，不直接持有用户 manager。
- `oracle_config.move` 是 Predict 对 oracle 的配置层，`oracle.move` 是核心价格状态。
