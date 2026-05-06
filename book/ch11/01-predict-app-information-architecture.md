# ch11-01 Predict 应用的信息架构

[返回本章](README.md)

## 先跑通场景

Predict 应用不能照搬现货交易终端。现货页面通常围绕订单簿、下单表单和成交历史组织；Predict 页面还要解释到期时间、区间边界、oracle 状态、vault 风险、费用和 settlement。用户不是在“按价格买币”，而是在购买某个 oracle 在某个 expiry 上落入某个区间的收益权。

所以信息架构要先回答风险语言，而不是先排组件：这个市场引用哪个 oracle，什么时候到期，区间边界是什么，价格是否 stale，买入后最大亏损和最大收益是多少，LP vault 是否承担了过高 max payout。

## 源码入口

- `packages/predict/sources/predict.move`、`predict_manager.move`、`vault/vault.move`：页面对象和资金路径。
- `packages/predict/simulations/src/shared.ts`：scenario/result schema 对信息架构的启发。
- `packages/predict/simulations/src/runtime.ts`：应用服务需要的对象 ID、PTB 和执行摘要。

## 从仿真到产品页面

Predict 前端至少需要五类视图：市场列表、市场详情、交易面板、我的头寸、LP vault。

市场列表不是简单卡片流。它至少要展示 underlying、expiry、strike 或区间、UP/DOWN/Range、oracle 状态、当前 fair price、fee 和 ask price。市场详情进一步展开 oracle spot、forward、basis、SVI 更新时间、Lazer 更新时间、staleness 状态和 settlement 状态。

交易面板必须把 fair value、fee、all-in cost、最大收益、最大亏损、break-even 和成交后 vault 风险讲清楚。我的头寸按 `RangeKey` 聚合 quantity，并从事件或自建索引恢复成本。LP vault 页面展示 vault balance、vault value、total MTM、total max payout、available withdrawal、PLP supply 和提款限速状态。

这里的产品结构要绑定 localnet 仿真和可签名 PTB，而不是假设已经存在完整 Predict Server。所有对象 ID 来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

> **版本旁白**：本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点，不能写成稳定产品依赖。

## Predict 应用判断

- 页面状态拆成 market catalog、oracle status、manager account、vault risk、simulation result 五块。
- 每个 market tile 都显示 oracle ID、expiry、strike range、quote asset 和版本状态。
- 不要依赖未完成 Predict Server；列表数据先来自配置、对象查询或本地仿真 state。
- 交易按钮旁边必须显示 oracle freshness 和 settlement 状态，否则用户无法判断失败原因。

## 动手检查

- 用户进入市场页前必须加载哪些 Predict 对象？
- 哪些字段属于链上可信状态，哪些只是 UI 缓存或仿真结果？
- 如果没有 Predict Indexer，市场列表和用户 PnL 各如何降级？
