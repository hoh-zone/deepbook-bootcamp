# ch12-02 安装和配置

[返回本章](README.md)

## 本节目标

- 安装并配置 Sui SDK、DeepBook SDK 和本章 TypeScript 骨架。
- 把网络、RPC、对象 ID、cap 和 coin type 放入集中配置。
- 为 Predict 增加迁移状态字段，避免误连未验证网络。

## 源码关联

- `book/ch12/code/s01-sdk-init/`：SDK 初始化模板。
- `scripts/config/constants.ts`：网络、cap、pool、manager 地址配置来源。
- `packages/predict/sources/*`：Predict 需要额外标注 package/object IDs 和版本状态。

## 正文

本章不安装依赖，只给出工程应声明的依赖方向：

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

运行时需要以下环境变量：

```bash
SUI_NETWORK=mainnet
SUI_RPC_URL=https://sui-mainnet.mystenlabs.com
SUI_ADDRESS=0x...
SUI_PRIVATE_KEY=suiprivkey...
GAS_OBJECT=0x...
```

前端不应读取 `SUI_PRIVATE_KEY`。前端只构造交易，交给钱包签名；后端只有在运维脚本、做市脚本或管理员 multisig 流程中才读取私钥或 keystore。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 开发要点

- 依赖安装后先跑类型检查或最小初始化脚本，确认 SDK 版本兼容。
- `.env` 或配置文件只存 RPC、network、object IDs，不存用户私钥。
- Predict 配置缺少迁移状态时只允许示例构造，不允许真实交易。

## 检查问题

- 安装完成后如何验证 `SuiClient` 和 DeepBook SDK 可用？
- 哪些配置应来自 `constants.ts` 风格的集中表？
- Predict 的 package ID 和版本状态为什么必须成对出现？
