# ch11-10 前端展示风险和收益

[返回本章](README.md)

## 本节目标

- 把交易者风险、LP 风险和 oracle 风险转化为前端字段。
- 展示 all-in ask、max payout、MTM freshness、withdraw limiter 和版本状态。
- 避免只用 UP/DOWN 标签隐藏 market key 细节。

## 源码关联

- `packages/predict/sources/vault/vault.move`：MTM、max payout、vault value。
- `packages/predict/sources/config/pricing_config.move`、`risk_config.move`：ask/risk 边界。
- `packages/predict/simulations/visualize.py`：风险与收益图表思路。

## 正文

交易卡片至少展示：成本、fee、最大 payout、最大亏损、到期条件、oracle 更新时间、报价是否 stale、vault utilization、ask bounds。LP 卡片至少展示：vault value、total MTM、total max payout、PLP share、withdrawal limiter、未结算 oracle 数量。

错误处理不要只显示 Move abort code。应用应把常见错误映射成可操作提示，例如 owner 不匹配、余额不足、quote asset 未启用、oracle stale、trading paused、ask price out of bounds、exposure 超限、MTM stale、withdrawal budget 不足。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## 开发要点

- 交易者视图展示 all-in ask、max loss、payout condition、oracle freshness。
- LP 视图展示 vault value、MTM、max payout、PLP NAV、limiter。
- 所有收益/风险图表标注 localnet simulation 或实际网络对象来源。

## 检查问题

- 交易者和 LP 分别最关心哪三个风险字段？
- oracle stale 与 vault exposure 超限在前端提示上有什么差异？
- 为什么只展示 UP/DOWN 会误导用户理解市场？
