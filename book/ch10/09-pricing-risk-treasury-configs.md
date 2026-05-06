# ch10-09 pricing config、risk config、treasury config

[返回本章](README.md)

## 本节目标

- 读懂 pricing、risk、treasury 三类配置分别约束价格、敞口和 quote 资产。
- 说明 fee 不是 bps，而是按 fair price 计算的绝对价格增量。
- 把配置校验映射到 mint、supply、withdraw 的错误提示。

## 源码关联

- `packages/predict/sources/config/pricing_config.move`：base fee、min fee、utilization fee、ask bounds。
- `packages/predict/sources/config/risk_config.move`：总敞口比例与 MTM freshness。
- `packages/predict/sources/config/treasury_config.move`：quote asset whitelist 和 6 位 decimals 校验。

## 正文

`pricing_config.move` 的 fee 不是 bps，而是 1e9 scale 的每单位绝对价格增量。`quote_fee_rate_from_fair_price` 使用 `base_fee * sqrt(price * (1 - price))`，再叠加 utilization fee，并受 `min_fee` 约束。`predict.move` 还会检查 all-in ask price 落在全局或 per-oracle ask bounds 内。

`risk_config.move` 只有两个字段：`max_total_exposure_pct` 和 `mtm_freshness_ms`。前者限制 vault 交易后总 MTM，后者用于 LP supply/withdraw 前检查所有未结算 oracle 的 MTM 是否足够新鲜。`treasury_config.move` 维护 quote asset 白名单，并要求 quote coin decimals 等于 `required_quote_decimals`，当前设计与 6 位 quote 单位对齐。

服务报价应同时返回 fair price、fee rate、all-in ask 和触发的配置来源。ask bounds 可以是全局或 oracle/asset 维度，UI 只显示最终是否可交易还不够，应该告诉用户是价格过高、利用率过高还是 quote asset 不被允许。

管理员更新配置时要通过 registry 或 Predict 管理入口，并在交易 bytes 里固定 gas、expiration 和 cap 对象。生产文档只能说“按当前发布版本入口执行”，不能假设迁移中的部署脚本已经稳定。

## 开发要点

- fee 计算文档统一写 1e9 price scale，不写 bps。
- mint 前同时检查 treasury whitelist、quote decimals、ask bounds 和 exposure。
- 配置更新记录 digest、版本和生效范围，便于回放报价差异。

## 检查问题

- base fee、min fee、utilization fee 分别解决什么问题？
- quote asset decimals 不等于 6 时为什么应直接拒绝？
- MTM freshness 和 max exposure 分别会拦截哪类操作？
