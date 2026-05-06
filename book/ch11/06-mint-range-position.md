# ch11-06 mint range position 的交易构造

[返回本章](README.md)

## 先跑通场景

这一节先把场景落到可执行流程：读者需要看到对象从哪里来，PTB 如何构造，交易前检查什么，失败后如何回到源码定位。

## 源码入口

- `packages/predict/sources/market_key/range_key.move`：有限区间 key。
- `packages/predict/sources/vault/vault.move`、`helper/strike_matrix.move`：range liability。
- `packages/predict/simulations/data/scenario_mar6_1000mints.csv`：多价格路径输入。

## 从仿真到交易

range position 与 binary position 的差别只是 lower/higher 都是有限 strike。应用应提供 strike grid 选择器，确保 lower、higher 都按 oracle tick size 对齐。`RangeKey::new` 只检查 `lower < higher` 和不是全覆盖区间，grid 合法性还要结合 oracle config 和 mint 路径校验。

区间市场适合表达“到期价格落在某个范围内”的观点。收益是固定二元 payout，不随落点深浅变化；不要把它展示成线性价差收益。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## Predict 应用判断

- range mint 必须校验 `lower < higher` 且二者在 strike grid 内。
- 有限区间的 UI 同时展示 payout 条件 `(lower, higher]` 和 settlement rule。
- dry run 后检查 vault liability、max payout 和用户 position quantity。

## 动手检查

- 有限 range position 与 UP/DOWN position 的 key 有什么不同？
- 为什么不能构造 `(-inf, +inf]`？
- range 越宽时 vault risk 和 fee 可能如何变化？
