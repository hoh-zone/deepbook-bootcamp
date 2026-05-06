# s03 arbitrage bot

本目录是套利和闪电贷组合交易的骨架。它不应该先追求复杂路径，第一版只验证“借出、交易、归还、利润检查”能在同一笔 PTB 内闭合。

## 套利前检查

- 路径价格差。
- DeepBook 手续费。
- 预估 slippage。
- Gas。
- 闪电贷归还条件。
- dry run 结果。

只有净利润覆盖失败风险时才提交交易。

## 建议命令

```bash
pnpm init
pnpm add @mysten/sui @mysten/deepbook-v3 decimal.js
pnpm add -D typescript tsx @types/node
pnpm exec tsx src/scan.ts
pnpm exec tsx src/submit.ts --dry-run
```

## 关键文件

```text
src/route.ts        # 路径和价格差计算
src/flashloan.ts    # borrow/return PTB 片段
src/profit.ts       # fee、gas、slippage 后净利润
src/submit.ts       # dry run 和提交
```

## 验收

- 不满足归还条件时 PTB 必须 dry run 失败。
- 净利润小于阈值时不提交。
- 所有成功交易保存 digest，并能在 `FlashLoanBorrowed` 事件或 indexer 表中反查。
