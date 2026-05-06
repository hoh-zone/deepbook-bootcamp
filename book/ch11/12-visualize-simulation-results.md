# ch11-12 仿真结果如何可视化

[返回本章](README.md)

## 本节目标

- 把 simulation `results.json` 转成 gas、延迟、成功率和 LP 风险图表。
- 说明 `visualize.py` 的输入输出和可扩展字段。
- 避免把仿真图表写成主网 SLA。

## 源码关联

- `packages/predict/simulations/src/shared.ts`：results schema。
- `packages/predict/simulations/visualize.py`：图表生成。
- `packages/predict/simulations/data/scenario_mar6_1000mints.csv`：输入场景。

## 正文

`shared.ts` 定义 `results_v2`：`summary.totalTxs`、`summary.byAction.{update_prices,update_svi,mint}` 和 `mints[]`。每个 action summary 包含 count、gas avg/min/max、wallMs avg/min/max；mint row 包含 wallMs、computationCost、storageCost、storageRebate、gasTotal。

运行方式：

```bash
cd deepbookv3
bash packages/predict/simulations/run.sh
```

已有 run 可恢复：

```bash
cd deepbookv3/packages/predict/simulations
bash run.sh --list
bash run.sh --resume <run-id> --sim
npm run analyze -- runs/<run-id>/artifacts/results.json
```

localnet 仿真输出适合比较 gas、延迟和 mint 执行数据，不提供 replay-derived trace profile。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## 开发要点

- 图表至少覆盖成功率、gas 分布、wallMs、vault value、MTM、PLP NAV。
- 读取 `results.json` 时保留失败 action 和 abort reason，不只画成功交易。
- 报告标题和注释明确来自 simulation results，而不是主网监控。

## 检查问题

- `results.json` 中哪些字段适合做 gas/latency 图？
- LP 收益曲线需要哪些 vault 和 fee 字段？
- 为什么仿真可视化不能当作生产 SLA？
