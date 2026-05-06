# s01-sdk-init

本示例是 DeepBook SDK 初始化模板，目标是把网络、RPC、sender、package/object IDs 和 SDK client 放在同一个可测试的入口中。

## 依赖方向

```json
{
  "dependencies": {
    "@mysten/deepbook-v3": "<pin a tested version>",
    "@mysten/sui": "<pin a tested version>"
  },
  "devDependencies": {
    "tsx": "<pin a tested version>",
    "typescript": "<pin a tested version>"
  }
}
```

运行命令方向：

```bash
pnpm tsx src/index.ts
```

## 环境变量

```bash
SUI_NETWORK=mainnet
SUI_RPC_URL=https://sui-mainnet.mystenlabs.com
SUI_ADDRESS=0x...
SUI_PRIVATE_KEY=suiprivkey... # 只给后端脚本使用，前端不要读取
GAS_OBJECT=0x...              # 管理员或 multisig 流程使用
```

## 配置骨架

配置形态参考 [scripts/config/constants.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/config/constants.ts)。不要在业务函数里散写 object ID。

```ts
export type SuiNetwork = "mainnet" | "testnet" | "devnet" | "localnet";

export type DeepBookRuntimeConfig = {
  network: SuiNetwork;
  rpcUrl: string;
  address: string;
  adminCap?: string;
  marginAdminCap?: string;
  marginMaintainerCap?: string;
  supplierCap?: string;
  balanceManagers: Record<string, { address: string }>;
};

export const configs: Record<string, Partial<DeepBookRuntimeConfig>> = {
  mainnet: {
    adminCap: "0x...",
    marginAdminCap: "0x...",
    supplierCap: "0x...",
    balanceManagers: {
      USER_MANAGER: { address: "0x..." },
    },
  },
  testnet: {
    adminCap: "0x...",
    balanceManagers: {},
  },
};
```

## SuiClient 初始化

继承式 `DeepBookClient` 初始化参考 [scripts/transactions/deepbookMarketMaker.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/deepbookMarketMaker.ts)：

```ts
import { SuiClient, getFullnodeUrl } from "@mysten/sui/client";
import { DeepBookClient } from "@mysten/deepbook-v3";

export function createDeepBookClient(config: DeepBookRuntimeConfig) {
  const suiClient = new SuiClient({
    url: config.rpcUrl || getFullnodeUrl(config.network),
  });

  const deepbook = new DeepBookClient({
    address: config.address,
    env: config.network === "localnet" ? "testnet" : config.network,
    client: suiClient,
    balanceManagers: config.balanceManagers,
    adminCap: config.adminCap,
  });

  return { suiClient, deepbook };
}
```

插件式初始化参考 [scripts/transactions/createPool.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/createPool.ts)：

```ts
import { deepbook } from "@mysten/deepbook-v3";
import { SuiGrpcClient } from "@mysten/sui/grpc";

export function createGrpcDeepBookClient(config: DeepBookRuntimeConfig) {
  return new SuiGrpcClient({
    baseUrl: config.rpcUrl,
    network: config.network,
  }).$extend(
    deepbook({
      address: config.address,
      adminCap: config.adminCap,
      marginAdminCap: config.marginAdminCap,
      marginMaintainerCap: config.marginMaintainerCap,
      balanceManagers: config.balanceManagers,
    }),
  );
}
```

## 对象校验

初始化后先校验关键对象存在：

```ts
export async function assertObjectExists(client: SuiClient, id: string, label: string) {
  if (!id || id === "0x") throw new Error(`${label} is not configured`);

  const object = await client.getObject({ id, options: { showType: true, showOwner: true } });
  if (!object.data) throw new Error(`${label} not found: ${id}`);

  return object.data;
}
```

生产环境至少检查 admin cap、margin cap、supplier cap、BalanceManager、Pool、Predict registry 这些对象是否属于当前网络。

## 交易骨架

```ts
import { Transaction } from "@mysten/sui/transactions";

export async function buildCreateBalanceManager(config: DeepBookRuntimeConfig) {
  const client = createGrpcDeepBookClient(config);
  const tx = new Transaction();
  client.deepbook.balanceManager.createAndShareBalanceManager()(tx);
  tx.setSender(config.address);
  return tx;
}
```

提交前必须 dry run。管理员交易可以参考 [scripts/utils/utils.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/utils/utils.ts) 的 `prepareMultisigTx` 设置 gas price、expiration、gas payment。
