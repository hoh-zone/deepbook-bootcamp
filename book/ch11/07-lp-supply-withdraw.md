# ch11-07 LP supply 和 withdraw

[返回本章](README.md)

## 本节目标

- 实现 LP supply/withdraw 的应用流程。
- 展示 PLP shares、vault value、MTM、max payout 和 rate limiter。
- 区分 LP 本地仿真收益曲线和真实主网收益承诺。

## 源码关联

- `packages/predict/sources/predict.move`：`supply<Quote>`、`withdraw<Quote>`。
- `packages/predict/sources/vault/plp.move`、`vault/vault.move`：PLP 和 vault value。
- `packages/predict/sources/helper/rate_limiter.move`：提款限速。

## 正文

LP supply 调用 `predict::supply<Quote>(predict, coin, clock)`，返回 PLP coin。仿真中 `supplyTx` 使用测试 DUSDC mint 后供应，并把 PLP 转回当前地址。withdraw 调用 `predict::withdraw<Quote>(predict, plp_coin, clock, ctx)`，源码会先检查所有 unsettled exposed oracle 的 MTM freshness，再按 PLP 占比计算可提 quote，并消耗 withdrawal limiter。

前端需要展示 `available_withdrawal(predict, clock)`，并在提款失败时区分 MTM stale、rate limited、vault balance 不足和 quote asset 不存在。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## 开发要点

- supply 前展示当前 vault value、total PLP、预估 shares 和 quote asset。
- withdraw 前检查 MTM freshness、limiter available、PLP balance 和未结算 oracle。
- 收益曲线同时拆分 LP fee 收益与 settlement liability。

## 检查问题

- 首次 supply 和后续 supply 的 PLP 计算有什么差异？
- 提款失败时如何区分 limiter、MTM stale 和 vault value 问题？
- LP 页面为什么必须展示 max payout 而不只展示 APR？
