# s04-repay-and-close

目标：实现部分还款、全部还款和平仓流程。

基础命令：

```bash
pnpm tsx index.ts --pool SUI_USDC --repay USDC --amount 100
pnpm tsx index.ts --pool SUI_USDC --repay USDC --all
pnpm tsx index.ts --pool SUI_USDC --close
```

链上入口：

- `margin_manager.move`：`repay_base`、`repay_quote`、`withdraw`。
- `pool_proxy.move`：`cancel_all_orders`、`withdraw_settled_amounts`、reduce-only 下单。

平仓建议流程：

1. 取消 manager 在 DeepBook Pool 的所有 open orders。
2. 结算 settled amounts。
3. 如果没有足够 debt asset，反向交易换出 debt asset。
4. 调 `repay_base` 或 `repay_quote`。
5. 债务清零后提取剩余 base、quote 和 DEEP。

错误提示：

- `EIncorrectMarginPool`：还款资产和当前债务池不匹配。
- `ERepayAmountTooLow`：还款金额过小。
- `ERepaySharesTooLow`：金额换算后的 shares 为 0。
- 余额不足：manager 中没有足够 debt asset。

输出应包含还款前 debt amount、repay shares、还款后 debt amount、剩余资产和最新风险率。
