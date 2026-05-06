# s04 Vault accounting

目标：用最小账本模拟 Predict vault 中的资金变化，帮助读者理解 `vault/vault.move`、`vault/plp.move` 和 `accounting/fee_reserve.move`。

## 输入

- LP supply 数量。
- 用户 mint position 的 quote 输入。
- ask price。
- fee reserve 增量。
- market settlement price。
- position payout。

## 输出

- vault quote balance。
- total PLP supply。
- fee reserve balance。
- outstanding max payout。
- settled 后可提现金额。

## 关键不变量

Predict vault 不是订单簿撮合池，也不是 MarginPool。它必须同时考虑 LP 资金、用户 position 的潜在 payout、已收 fee 和结算后的 redeem。示例实现时要把这些账本分开，不要只维护一个 `balance`。

建议运行方式：

```bash
pnpm init
pnpm add -D typescript tsx vitest @types/node
pnpm exec vitest run
```

测试至少覆盖：首次 LP supply、用户 mint、fee reserve 增加、settlement payout、LP withdraw 五个状态变化。
