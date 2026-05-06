# s02-pool-object-query

目标：查询链上 DeepBook Pool 对象，确认对象类型、版本化 inner 和池参数。

环境变量：

```bash
export SUI_RPC_URL=https://fullnode.mainnet.sui.io:443
export POOL_ID=0x...
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
const object = await client.getObject({
  id: process.env.POOL_ID!,
  options: { showType: true, showContent: true, showOwner: true },
});
console.dir(object, { depth: null });
```

核对点：

- object type 是否是 `deepbook::pool::Pool<BaseAsset, QuoteAsset>`。
- owner 是否为 shared。
- content 中 `inner` 是否是 versioned 对象引用。
- 池参数需要结合 SDK、公开对象解析或 Move view/query 模块读取，不要假设 JSON 中直接展开全部 `PoolInner`。

生产注意事项：主网 Pool 对象版本和 package ID 可能随协议升级变化。交易构造代码应从配置或 SDK 读取常量，并对 `EPackageVersionDisabled` 类 abort 做明确提示。
