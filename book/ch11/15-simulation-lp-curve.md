# ch11-15 本章实战：仿真价格路径和 LP 收益曲线

[返回本章](README.md)

## 先跑通场景

这一节从 Predict 的用户动作切入：先确认用户或 operator 要提交哪些对象和参数，再回到源码看市场状态如何被约束。

## 源码入口

- `book/ch11/code/s05-simulation-report/`：报告骨架。
- `packages/predict/simulations/src/shared.ts`、`visualize.py`：results 与图表。
- `packages/predict/simulations/src/sim.ts`：价格路径与 LP 状态变化。

## 从仿真到交易

基于现有 simulation runtime，可以扩展 CSV schema，加入 LP supply/withdraw 和 redeem action。结果报告除了 gas/latency，还应计算 vault balance、vault value、total MTM、total max payout、PLP NAV 和 fee reserve 增长。

不要把仿真结果当成收益承诺。SVI 参数、basis bounds、用户 mint 分布、oracle staleness 和 withdrawal limiter 都会显著改变 LP 曲线。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## Predict 应用判断

- 扩展 CSV schema 时明确 action、timestamp、spot、basis、quantity、lp action。
- 报告同时输出 vault balance、vault value、MTM、max payout、PLP NAV 和 fee reserve。
- 收益曲线标注路径假设和 localnet 限制。

## 动手检查

- 价格路径如何影响 binary/range position 的 MTM？
- LP 收益曲线中哪些变化来自 fee，哪些来自 liability？
- 新增 redeem/withdraw action 后 results schema 要补哪些字段？
