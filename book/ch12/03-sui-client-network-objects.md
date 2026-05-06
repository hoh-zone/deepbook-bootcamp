# ch12-03 SuiClient、network 和对象配置

[返回本章](README.md)

## 先定封装边界

读这一节时把自己当成 SDK 维护者：封装应该暴露业务意图，隐藏重复参数，同时保留 dry run 和错误定位能力。

## 源码入口

- `scripts/config/constants.ts`：admin cap、margin cap、supplier cap、market maker 配置形态。
- `scripts/utils/utils.ts`：signer、gas payment 和 dry run 工具。
- `packages/predict/sources/registry.move`：Predict registry/predict object ID 的配置来源。

## SDK 读法

DeepBook 交易不能把对象 ID 写散在业务函数里。配置层至少包含：

- `network`：`mainnet`、`testnet`、`devnet` 或 `localnet`。
- `rpcUrl`：优先来自环境变量，默认可以回退到 Sui 官方 fullnode URL。
- `address`：构造交易的发送者，也是 SDK 初始化时的 `address`。
- `adminCap`、`marginAdminCap`、`marginMaintainerCap`：管理员路径使用。
- `balanceManagers`：业务账户和 `BalanceManager` object ID 的映射。
- `poolKey`、`coinKey`：业务传 `"SUI_DBUSDC"`、`"USDC"` 这类 key，不直接拼 Move type。

`scripts/config/constants.ts` 中的 `adminCapID`、`marginAdminCapID`、`supplierCapID`、`marketMakerID` 展示了主网和测试网配置表的形态。测试网字段为空时，服务层必须显式报错，而不是默默发送无效对象 ID。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- `SuiClient` 初始化显式接收 network、rpcUrl、address，不从全局隐式读取。
- 对象配置按网络分组，mainnet/testnet/localnet 不能互相复用。
- 读取共享对象后校验 type、owner/shared、object version 和 package 来源。

## 动手检查

- 同一个 object ID 放错网络会导致什么失败？
- 服务层何时应该抛 `CONFIG_MISSING`？
- Predict oracle ID 和 predict object ID 应在哪一层校验？
