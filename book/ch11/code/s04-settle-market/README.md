# s04 Settle market

目标：描述 Predict market 到期后的 settlement、redeem 和 UI 状态切换。

## 流程

1. oracle 提供 settlement price。
2. market 从 trading 状态进入 settled 状态。
3. 用户 position 根据 payoff 规则计算 payout。
4. 用户 redeem position。
5. LP 根据 vault 状态继续 withdraw 或承担 payout 后的净值变化。

## 开发注意

结算页要显示 oracle ID、settlement price、expiry、position side、payout 和交易 digest。不要在 settlement price 未确认前展示确定收益。

