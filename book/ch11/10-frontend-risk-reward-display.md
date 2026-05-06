# ch11-10 前端展示风险和收益

[返回本章](README.md)

## 先跑通场景

这一节先把场景落到可执行流程：读者需要看到对象从哪里来，PTB 如何构造，交易前检查什么，失败后如何回到源码定位。

## 源码入口

- `packages/predict/sources/vault/vault.move`：MTM、max payout、vault value。
- `packages/predict/sources/config/pricing_config.move`、`risk_config.move`：ask/risk 边界。
- `packages/predict/simulations/visualize.py`：风险与收益图表思路。

## 从仿真到交易

交易卡片至少展示：成本、fee、最大 payout、最大亏损、到期条件、oracle 更新时间、报价是否 stale、vault utilization、ask bounds。LP 卡片至少展示：vault value、total MTM、total max payout、PLP share、withdrawal limiter、未结算 oracle 数量。

错误处理不要只显示 Move abort code。应用应把常见错误映射成可操作提示，例如 owner 不匹配、余额不足、quote asset 未启用、oracle stale、trading paused、ask price out of bounds、exposure 超限、MTM stale、withdrawal budget 不足。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## Predict 应用判断

- 交易者视图展示 all-in ask、max loss、payout condition、oracle freshness。
- LP 视图展示 vault value、MTM、max payout、PLP NAV、limiter。
- 所有收益/风险图表标注 localnet simulation 或实际网络对象来源。

## 动手检查

- 交易者和 LP 分别最关心哪三个风险字段？
- oracle stale 与 vault exposure 超限在前端提示上有什么差异？
- 为什么只展示 UP/DOWN 会误导用户理解市场？
