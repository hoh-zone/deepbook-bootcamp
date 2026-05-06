# s04 margin dashboard

本目录是 Margin 风控面板的 UI 蓝图。它要把仓位风险讲清楚，而不是只展示余额。

## 核心卡片

- collateral
- debt
- risk ratio
- interest accrued
- borrow limit
- liquidation threshold

每次用户操作前展示操作后的风险率。

## 建议命令

```bash
pnpm create vite . --template react-ts
pnpm install
pnpm add @mysten/sui @mysten/deepbook-v3 decimal.js
pnpm dev
```

## 数据源

- 链上对象：MarginManager、MarginPool、oracle price object。
- Server/Indexer：借款、还款、清算、利息和历史交易。
- 本地计算：风险率、可借额度、清算距离、操作后风险率。

## 验收

- 每个借款或提现按钮都先展示操作后风险率。
- oracle stale 或 indexer lag 时禁止给出“健康”结论。
- 清算阈值、当前风险率和安全缓冲使用同一套精度计算。
