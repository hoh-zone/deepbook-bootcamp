# ch10-02 预测市场、二元期权、区间市场和 LP vault

[返回本章](README.md)

## 本节目标

- 解释 Predict 如何用 `RangeKey` 表达二元 UP/DOWN 和有限区间市场。
- 说明用户头寸、LP vault、PLP 和结算 payout 之间的资金关系。
- 识别预测市场 UI 里必须展示的 oracle、expiry、strike 和 vault 风险字段。

## 源码关联

- `packages/predict/sources/market_key/range_key.move`：`(oracle, expiry, lower, higher]` 的 market key 定义。
- `packages/predict/sources/predict.move`：`mint`、`redeem`、`supply`、`withdraw` 的外部入口。
- `packages/predict/sources/vault/vault.move` 与 `vault/plp.move`：LP 资金池、敞口和 PLP 份额。

## 正文

Predict 的交易单位是一个在到期时支付 0 或 `quantity` 的区间头寸。`range_key.move` 把头寸定义为 `(oracle_id, expiry, lower_strike, higher_strike)`，结算规则是价格落在 `(lower, higher]` 内则支付 `quantity`，否则支付 0。二元 UP 市场可以表示为 `(strike, +inf]`，DOWN 市场可以表示为 `(-inf, strike]`，更宽的区间市场则直接设置两个有限 strike。

LP vault 是交易对手方。用户 mint 时从 `PredictManager` 扣 quote，vault 接收 fair value 和 LP fee，同时记录潜在赔付；用户 live redeem 时 vault 支付当前 fair value 扣 fee；到期后 redeem 按 settlement price 精确支付。LP 通过 `predict.move::supply` 获得 PLP，通过 `withdraw` 烧毁 PLP 取回 vault value 的份额。

PTB 构造上，二元市场和区间市场的差异只体现在传给 `mint<Quote>` 的 `RangeKey`。UP 可以使用正无穷 sentinel 作为 upper bound，DOWN 可以使用负无穷 sentinel 作为 lower bound；有限区间则传两个 strike。无论 UI 标签怎么写，链上 key 都必须携带 oracle ID 和 expiry。

LP 不是被动收取固定利息，而是在 vault 中承担所有未结算区间的对手方风险。服务层报价时应同时返回 fair value、fee、all-in ask、vault MTM、max payout 和是否满足 exposure limit。

## 开发要点

- 市场卡片必须展示 oracle、expiry、strike range 和 quote asset，而不是只显示 UP/DOWN。
- mint 前检查 `PredictManager` 余额、oracle 新鲜度、ask bounds 和 vault exposure。
- LP 页面同时展示 PLP NAV、MTM、max payout、withdraw limiter 和 settlement 状态。

## 检查问题

- 为什么同一个 strike 在不同 oracle 或 expiry 下不是同一个市场？
- 用户 live redeem 和 settled redeem 分别由 vault 支付什么金额？
- LP vault 的 `balance`、`total_mtm`、`total_max_payout` 对 UI 风险提示有什么影响？
