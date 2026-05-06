# ch11-11 仿真脚本如何构造市场路径

[返回本章](README.md)

## 本节目标

- 读懂 simulation runtime 如何构造 setup、scenario、oracle refresh 和 mint。
- 把 CSV 场景映射到 PTB 执行、重试和 gas 汇总。
- 说明 localnet 结果只能用于开发评估。

## 源码关联

- `packages/predict/simulations/run.sh`：setup、sim、resume、analyze。
- `packages/predict/simulations/src/runtime.ts`：client、PTB、retry、gas 汇总。
- `packages/predict/simulations/src/sim.ts`：scenario 执行和结果写入。

## 正文

`sim.ts::setupSimulation` 完成 localnet 初始状态：finalize DUSDC currency、create Predict、create OracleCap、set BTC feed id、放宽 basis bounds、create oracle、activate oracle、supply vault、create manager、deposit manager，并把 `predictId`、`oracleId`、`oracleCapId`、`managerId`、`expiry` 写入 `artifacts/state.json`。

`executeScenario` 读取 CSV。若连续三行是 `update_prices -> update_svi -> mint`，它会合成一个 `refreshOracleAndMintTx`，在一个 PTB 中更新 basis、更新 SVI、构造 `RangeKey` 并 mint。其他行则分别执行 update 或 mint。每个动作记录 wall time 和 gas。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## 开发要点

- `run.sh` 阶段拆成 setup、sim、resume、analyze，并保留 artifacts state。
- scenario CSV 行映射为 oracle refresh、mint 和结果记录，不写成通用撮合引擎。
- gas 和 wallMs 汇总保存 action、digest、status、abort 信息。

## 检查问题

- setup state 中哪些 object ID 会被后续 scenario 复用？
- resume 如何避免重复发布或重复创建对象？
- localnet gas/wallMs 能回答什么问题，不能回答什么问题？
