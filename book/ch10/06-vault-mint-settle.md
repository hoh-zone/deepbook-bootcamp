# ch10-06 `vault/vault.move` 的资金池、mint 和 settle

[返回本章](README.md)

## 本节目标

- 解释 `Vault` 如何记录 quote 余额、区间库存、MTM 和最大赔付。
- 理解 mint、live redeem、settled redeem 对 vault liability 的不同影响。
- 把 LP 风险指标映射成应用需要展示和 dry run 检查的字段。

## 源码关联

- `packages/predict/sources/vault/vault.move`：`Vault`、`insert_live_range`、`remove_live_range`、`remove_settled_range`、`vault_value`。
- `packages/predict/sources/helper/strike_matrix.move`：区间库存、MTM、worst-case payout。
- `packages/predict/sources/config/risk_config.move`：`max_total_exposure_pct` 与 `mtm_freshness_ms`。

## 正文

`Vault` 用 `Bag` 保存不同 quote asset 的实际 `Balance<T>`，并用 `balance: u64` 维护统一 quote 单位余额。每个 oracle 对应一个 `StrikeMatrix`，用于记录所有区间头寸的边界变化。`insert_live_range` 在 mint 后插入区间并刷新 live MTM；`remove_live_range` 在 live redeem 时移除区间；`remove_settled_range` 在 settled redeem 时按 settlement 减少 liability。

风险核心是三个数：`total_mtm` 表示当前 mark-to-market liability，`total_max_payout` 表示最坏结算赔付，`vault_value()` 返回 `balance - total_mtm`。`assert_total_exposure(max_total_pct)` 用 `total_mtm <= balance * max_total_pct` 限制交易后敞口。`compact_settled_oracle_if_needed` 会把已结算 oracle 的密集矩阵压缩成固定 liability 状态，减少长期存储压力。

vault 的核心不是单个余额，而是“余额减去未结算负债”的动态净值。mint 增加 quote payment，也增加区间 liability；live redeem 反向移除 live range 并支付 fair value；settled redeem 按 settlement price 移除固定赔付。

服务封装里应把 `vault_value` 作为 LP NAV 的基础，把 `total_max_payout` 作为压力场景展示。dry run 如果因为 exposure limit 或 MTM freshness 失败，应提示用户这是 vault 风险约束，不是钱包余额不足。

## 开发要点

- 所有 LP 估值都使用 `vault_value()` 思路，而不是裸 `balance`。
- mint 后同时记录 payment、fee、MTM 增量和 max payout 增量。
- settled oracle compact 是存储优化，不改变用户应得 payout。

## 检查问题

- `total_mtm` 和 `total_max_payout` 分别回答什么风险问题？
- 为什么 LP withdraw 需要 MTM 新鲜度检查？
- settled redeem 对 vault 的 balance 和 liability 各有什么影响？
