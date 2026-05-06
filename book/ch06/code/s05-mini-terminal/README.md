# s05 最小 CLI 交易终端

目标：把池子列表、余额、订单簿、下单、撤单、成交查询串成一个命令行工具。

建议命令：

```bash
pnpm install
pnpm tsx terminal.ts markets
pnpm tsx terminal.ts balances --manager 0x...
pnpm tsx terminal.ts book --pool 0x...
pnpm tsx terminal.ts limit --side bid --price 100000 --quantity 1000000
pnpm tsx terminal.ts market --side ask --quantity 1000000
pnpm tsx terminal.ts cancel --order-id 123
```

工程约定：

- 所有金额内部使用 bigint。
- 所有交易先 dry run，再请求签名。
- 事件确认后再更新本地订单状态。
- abort code 映射到明确错误文案。
