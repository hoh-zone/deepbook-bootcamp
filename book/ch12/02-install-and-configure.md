# ch12-02 安装和配置

[返回本章](README.md)

## 先定封装边界

SDK 小节先看封装边界：好的服务层不是隐藏 Move，而是减少对象、精度、权限和网络配置错误，同时保留 dry run 与错误定位能力。

## 源码入口

- `book/ch12/code/s01-sdk-init/`：SDK 初始化模板。
- `scripts/config/constants.ts`：网络、cap、pool、manager 地址配置来源。
- `packages/predict/sources/*`：Predict 需要额外标注 package/object IDs 和版本状态。

## SDK 读法

SDK 集成要先能运行一个最小初始化脚本，再谈交易封装。建议把本章示例放进独立的 TypeScript 工作区，先固定依赖，再把网络、package ID、pool ID 和钱包签名方式分层管理。

最小依赖：

```bash
pnpm init
pnpm add @mysten/deepbook-v3 @mysten/sui
pnpm add -D typescript tsx @types/node
```

`package.json` 至少保留一个可执行入口：

```json
{
  "type": "module",
  "scripts": {
    "check:sdk": "tsx src/check-sdk.ts"
  },
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

运行时配置不要混在代码里。主网、测试网、本地 sandbox 至少拆成三组环境变量：

```bash
SUI_NETWORK=mainnet
SUI_RPC_URL=https://sui-mainnet.mystenlabs.com
SUI_ADDRESS=0x...
DEEPBOOK_CONFIG_PATH=./config/mainnet.json
```

本地 sandbox 可以直接指向 ch16 的服务：

```bash
SUI_NETWORK=localnet
SUI_RPC_URL=http://localhost:9000
DEEPBOOK_SERVER_URL=http://localhost:9008
DEEPBOOK_MANIFEST_URL=http://localhost:9009/manifest
```

后端运维脚本或机器人可以使用私钥，但要和用户交易隔离：

```bash
SUI_PRIVATE_KEY=suiprivkey...
GAS_OBJECT=0x...
```

前端不应读取 `SUI_PRIVATE_KEY`。前端只构造交易，交给钱包签名；后端只有在运维脚本、做市脚本或管理员 multisig 流程中才读取私钥、keystore 或 multisig cap。

最小验证脚本不应该一上来下单。先确认 RPC、地址格式、配置文件和 SDK import 都可用：

```ts
// src/check-sdk.ts
import { SuiClient, getFullnodeUrl } from "@mysten/sui/client";

const rpcUrl = process.env.SUI_RPC_URL ?? getFullnodeUrl("testnet");
const client = new SuiClient({ url: rpcUrl });

const chain = await client.getChainIdentifier();
console.log({ rpcUrl, chain });
```

运行：

```bash
SUI_RPC_URL=http://localhost:9000 pnpm check:sdk
```

如果本地 sandbox 尚未启动，这一步应该失败在 RPC 连接，而不是失败在 DeepBook 配置。这样才能把“网络不可用”和“交易参数错误”分开排查。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- 依赖安装后先跑类型检查或最小初始化脚本，确认 SDK 版本兼容。
- `.env` 或配置文件只存 RPC、network、object IDs，不存用户私钥。
- Predict 配置缺少迁移状态时只允许示例构造，不允许真实交易。
- sandbox 重置后重新读取 manifest，不复用旧 package ID 或 pool ID。

## 动手检查

- 安装完成后如何验证 `SuiClient` 和 DeepBook SDK 可用？
- 哪些配置应来自 `constants.ts` 风格的集中表？
- Predict 的 package ID 和版本状态为什么必须成对出现？
