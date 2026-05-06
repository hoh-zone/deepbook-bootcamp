# ch11-01 Predict 应用的信息架构

[返回本章](README.md)

## 本节目标

- 设计 Predict 应用的信息架构，把 market、oracle、manager、vault 和 PLP 风险分层展示。
- 把 ch10 的链上对象映射到前端页面、后端服务和本地仿真数据。
- 明确当前应用示例是 localnet/simulation 骨架，不是稳定主网 Predict Server。

## 源码关联

- `packages/predict/sources/predict.move`、`predict_manager.move`、`vault/vault.move`：页面对象和资金路径。
- `packages/predict/simulations/src/shared.ts`：scenario/result schema 对信息架构的启发。
- `packages/predict/simulations/src/runtime.ts`：应用服务需要的对象 ID、PTB 和执行摘要。

## 正文

Predict 前端至少需要五类视图：市场列表、市场详情、交易面板、我的头寸、LP vault。市场列表展示 underlying、expiry、strike 或区间、UP/DOWN/Range、oracle 状态、当前 fair price、fee 和 ask price。市场详情展示 oracle spot、forward、basis、SVI 更新时间、Lazer 更新时间、staleness 状态和 settlement 状态。

交易面板必须把 fair value、fee、all-in cost、最大收益、最大亏损、break-even 和成交后 vault 风险展示清楚。我的头寸按 `RangeKey` 聚合 quantity，并从事件或自建索引恢复成本。LP vault 页面展示 vault balance、vault value、total MTM、total max payout、available withdrawal、PLP supply 和提款限速状态。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## 开发要点

- 页面状态拆成 market catalog、oracle status、manager account、vault risk、simulation result 五块。
- 每个 market tile 都显示 oracle ID、expiry、strike range、quote asset 和版本状态。
- 不要依赖未完成 Predict Server；列表数据先来自配置、对象查询或本地仿真 state。

## 检查问题

- 用户进入市场页前必须加载哪些 Predict 对象？
- 哪些字段属于链上可信状态，哪些只是 UI 缓存或仿真结果？
- 如果没有 Predict Indexer，市场列表和用户 PnL 各如何降级？
