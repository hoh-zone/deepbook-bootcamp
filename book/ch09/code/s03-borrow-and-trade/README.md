# s03-borrow-and-trade

目标：构造“借款并交易”的 PTB，覆盖杠杆买入和杠杆卖出。

杠杆买入示例：

```bash
pnpm tsx index.ts --pool SUI_USDC --borrow USDC --amount 500 --side buy --quantity 100 --order market
```

杠杆卖出示例：

```bash
pnpm tsx index.ts --pool SUI_USDC --borrow SUI --amount 100 --side sell --quantity 100 --order market
```

链上入口：

- `margin_manager.move`：`borrow_base`、`borrow_quote`。
- `pool_proxy.move`：`place_market_order`、`place_limit_order`、`withdraw_settled_amounts`。

交易前检查：

- Pool 已启用 Margin。
- 目标 MarginPool 已授权该 DeepBook Pool 借款。
- manager 没有其它 MarginPool 债务。
- 借款金额大于 `min_borrow`。
- 借款后风险率高于 `min_borrow_risk_ratio`。
- 市价单有足够订单簿流动性，effective price 能通过 `registry.assert_price`。

输出字段：

- 借款资产和金额。
- 预估成交价格和滑点。
- borrow 前后风险率。
- trade 后风险率。
- dry run abort code 映射。
