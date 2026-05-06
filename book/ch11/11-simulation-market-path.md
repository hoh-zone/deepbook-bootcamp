# ch11-11 仿真脚本如何构造市场路径

[返回本章](README.md)

## 先跑通场景

这一节从 Predict 的用户动作切入：先确认用户或 operator 要提交哪些对象和参数，再回到源码看市场状态如何被约束。

## 源码入口

- `packages/predict/simulations/run.sh`：setup、sim、resume、analyze。
- `packages/predict/simulations/src/runtime.ts`：client、PTB、retry、gas 汇总。
- `packages/predict/simulations/src/sim.ts`：scenario 执行和结果写入。

## 从仿真到交易

`sim.ts::setupSimulation` 完成 localnet 初始状态：finalize DUSDC currency、create Predict、create OracleCap、set BTC feed id、放宽 basis bounds、create oracle、activate oracle、supply vault、create manager、deposit manager，并把 `predictId`、`oracleId`、`oracleCapId`、`managerId`、`expiry` 写入 `artifacts/state.json`。

`executeScenario` 读取 CSV。若连续三行是 `update_prices -> update_svi -> mint`，它会合成一个 `refreshOracleAndMintTx`，在一个 PTB 中更新 basis、更新 SVI、构造 `RangeKey` 并 mint。其他行则分别执行 update 或 mint。每个动作记录 wall time 和 gas。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## Predict 应用判断

- `run.sh` 阶段拆成 setup、sim、resume、analyze，并保留 artifacts state。
- scenario CSV 行映射为 oracle refresh、mint 和结果记录，不写成通用撮合引擎。
- gas 和 wallMs 汇总保存 action、digest、status、abort 信息。

## 动手检查

- setup state 中哪些 object ID 会被后续 scenario 复用？
- resume 如何避免重复发布或重复创建对象？
- localnet gas/wallMs 能回答什么问题，不能回答什么问题？
