# ch10-07 `vault/plp.move` 的 LP 份额和收益

[返回本章](README.md)

## 本节目标

- 理解 PLP 是 Predict vault value 的份额凭证，而不是固定收益 token。
- 说明 supply/withdraw 如何铸造和烧毁 PLP，以及 fee 如何影响 LP NAV。
- 识别 LP 退出时的 MTM、rate limiter 和 settlement 风险。

## 源码关联

- `packages/predict/sources/vault/plp.move`：`PLP` token 注册、6 位精度和 treasury cap。
- `packages/predict/sources/predict.move`：`supply<Quote>`、`withdraw<Quote>`。
- `packages/predict/sources/helper/rate_limiter.move` 与 `vault/vault.move`：提款限速和 vault value。

## 正文

`plp.move` 注册 `PLP` token，精度为 6，symbol 为 `PLP`。PLP 不是 claim 某个具体 coin 的简单凭证，而是 Predict vault value 的份额。mint 交易中的 LP fee 会留在 vault，因此会提升 LP NAV；protocol 和 insurance fee 会进入 `FeeReserve`，不计入 LP vault value。

LP 退出时的风险点有三个：未结算敞口必须有新鲜 MTM；提款可能受 `RateLimiter` 限制；极端情况下 settlement payout 会把 `vault_value` 压低，所以前端不能只展示余额，必须展示 `total_mtm`、`total_max_payout`、available withdrawal 和未结算 oracle 列表。

首次 supply 可以按 amount 铸造 PLP，后续 supply 要按当前 vault value 与 total PLP 供应量计算份额。这样 LP fee 留在 vault 时，老 LP 的 NAV 会提高，新 LP 不能用原始 1:1 价格稀释已有收益。

提款 PTB 应先 dry run，读取失败是否来自 rate limiter、MTM freshness 或 vault value 不足。前端不要只给“提款失败”，而要显示当前 available withdrawal、下一次 refill 时间和未结算市场列表。

## 开发要点

- PLP 展示使用 6 位精度，并说明它代表 vault 净值份额。
- LP fee、protocol fee、insurance fee 分别展示，避免把全部 fee 记入 LP 收益。
- 提款前检查 limiter 状态和所有 live oracle 的 MTM 更新时间。

## 检查问题

- 为什么 LP fee 留在 vault 会提高 PLP NAV？
- rate limiter disabled 和 enabled 时 withdraw 行为有什么差异？
- 哪些风险会让 PLP 持有人不能按当前余额立即退出？
