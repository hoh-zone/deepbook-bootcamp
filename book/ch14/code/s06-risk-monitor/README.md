# s06 risk monitor

本目录是 DeepBook 应用风险监控服务的蓝图。它不参与撮合，但决定机器人和前端在异常时是否继续交易。

## 监控项

- RPC 延迟
- Indexer checkpoint lag
- Server 5xx
- 订单失败率
- 做市库存
- Margin 风险率
- 数据库延迟

风控触发时策略系统要能停止发单。

## 建议命令

```bash
pnpm init
pnpm add @mysten/sui prom-client
pnpm add -D typescript tsx @types/node
pnpm exec tsx src/monitor.ts
```

## 告警阈值示例

```bash
MAX_RPC_LATENCY_MS=1000
MAX_CHECKPOINT_LAG=20
MAX_SERVER_5XX_RATE=0.05
MAX_ORDER_FAILURE_RATE=0.10
```

## 验收

- RPC、Indexer、Server 任一核心依赖异常时发出结构化告警。
- 做市或套利策略能读取风险状态并停止发单。
- 每个告警包含网络、pool、checkpoint、最近 digest 和建议处理动作。
