# s02 market maker bot

本目录是做市机器人骨架。真实策略必须先在 ch16 sandbox 里跑通，再迁移到测试网；主网策略需要独立风控、限额和 key 管理。

## 主循环

1. 读取 mid price 和订单簿。
2. 计算 bid/ask 报价。
3. 检查库存和最大风险。
4. 撤掉过期订单。
5. 提交新订单。
6. 记录 digest 和错误。

第一版必须设置最大库存、最大挂单数和失败熔断。

## 建议命令

```bash
pnpm init
pnpm add @mysten/sui @mysten/deepbook-v3 decimal.js
pnpm add -D typescript tsx @types/node
pnpm exec tsx src/main.ts
```

## 最小配置

```bash
SUI_RPC_URL=http://localhost:9000
DEEPBOOK_SERVER_URL=http://localhost:9008
DEEPBOOK_MANIFEST_URL=http://localhost:9009/manifest
MAX_BASE_INVENTORY=100
MAX_QUOTE_INVENTORY=1000
MAX_OPEN_ORDERS=20
```

## 验收

- 每次报价前先 dry run。
- 每笔提交记录 digest、pool、side、price、quantity。
- 连续失败达到阈值后停止发单，只保留撤单和告警。
