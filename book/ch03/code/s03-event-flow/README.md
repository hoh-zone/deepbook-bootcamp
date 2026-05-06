# s03-event-flow

目标：按交易 digest 解析 DeepBookV3 事件，复原一次下单的事件链。

环境变量：

```bash
export SUI_RPC_URL=https://fullnode.mainnet.sui.io:443
export TX_DIGEST=...
```

建议依赖：

```bash
pnpm init
pnpm add @mysten/sui
pnpm add -D tsx typescript
```

最小查询思路：

```ts
import { SuiClient, getFullnodeUrl } from '@mysten/sui/client';

const client = new SuiClient({ url: process.env.SUI_RPC_URL ?? getFullnodeUrl('mainnet') });
const tx = await client.getTransactionBlock({
  digest: process.env.TX_DIGEST!,
  options: { showEvents: true, showEffects: true, showBalanceChanges: true },
});

for (const event of tx.events ?? []) {
  if (event.type.includes('deepbook')) {
    console.log(event.type, event.parsedJson);
  }
}
```

重点事件：

- `PoolCreated`：池创建参数。
- `OrderInfo`：一次 taker 订单的完整生命周期摘要。
- `OrderPlaced`：剩余数量进入订单簿。
- `OrderFilled`：maker/taker 成交明细。
- `OrderFullyFilled`：订单完全成交。
- `OrderCanceled` / `OrderExpired`：订单被取消或过期。
- `FlashLoanBorrowed`：DeepBookV3 闪电贷借出事件。

生产注意事项：事件顺序来自同一交易执行结果，但 indexer 落库可能异步。后端服务应按 checkpoint/digest/idempotency 去重，而不是只按时间戳排序。
