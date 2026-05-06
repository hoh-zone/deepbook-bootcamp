# s03-first-query 首次查询

本示例查询一个 DeepBook 池子对象。你需要先准备同一网络上的池子对象 ID。

## 环境变量

```bash
export NETWORK=mainnet
export RPC_URL=https://fullnode.mainnet.sui.io:443
export POOL_ID=0x...
```

`POOL_ID` 必须是 `Pool<BaseAsset, QuoteAsset>` 对象。池子对象由 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 中的 `Pool` 结构表示。

## CLI 查询

```bash
sui client object "$POOL_ID" --json
```

检查点：

- `objectId` 等于 `POOL_ID`。
- `type` 中包含 DeepBook package、`pool::Pool`、base asset、quote asset。
- `content` 能显示对象字段或 `Versioned` 外层信息。

## TypeScript 查询方向

```bash
pnpm add @mysten/sui tsx
pnpm tsx first-query.ts
```

`first-query.ts` 的核心逻辑：

```ts
import { SuiClient } from '@mysten/sui/client';

const client = new SuiClient({ url: process.env.RPC_URL! });
const objectId = process.env.POOL_ID!;

const object = await client.getObject({
  id: objectId,
  options: { showType: true, showContent: true, showOwner: true },
});

console.dir(object, { depth: null });
```

## 常见失败

- `ObjectNotFound`：`POOL_ID` 和 `RPC_URL` 不在同一网络。
- `showContent` 为空：检查 RPC 返回是否有错误字段。
- 查询成功但无法理解字段：先阅读 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 `Pool` 和 `PoolInner`。
