# ch12-04 DeepBookClient 初始化

[返回本章](README.md)

## 先定封装边界

SDK 小节先看封装边界：好的服务层不是隐藏 Move，而是减少对象、精度、权限和网络配置错误，同时保留 dry run 与错误定位能力。

## 源码入口

- `scripts/transactions/deepbookMarketMaker.ts`：`DeepBookClient` 类实例路径。
- `scripts/transactions/createPool.ts`：`SuiGrpcClient().$extend(deepbook(...))` 插件路径。
- `packages/deepbook/sources/pool.move`、`balance_manager.move`：客户端最终绑定的 Move 入口。

## SDK 读法

`scripts/transactions/deepbookMarketMaker.ts` 的核心初始化是：

```ts
const client = new DeepBookClient({
  address,
  env: "mainnet",
  client: new SuiClient({ url: getFullnodeUrl("mainnet") }),
  balanceManagers,
  adminCap,
});
```

继承式封装适合把签名能力和交易构造放在同一个类中。它的风险是容易让服务层托管用户私钥，所以应只用于后端自有账户。面向用户的钱包交易时，服务层应返回 `Transaction` 或交易 bytes，由钱包签名提交。

> **工程提醒**：SDK 初始化不是把 RPC URL 填进去就结束。你要明确签名者是谁、交易由谁提交、`BalanceManager` 属于谁、哪些对象来自配置，哪些对象来自用户选择。这个边界不清，后面所有 Move 调用都会变成权限和对象错误。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- 继承式 client 适合做市脚本，插件式 client 适合按模块调用。
- 初始化时把 signer、client、balanceManagers、pools 依赖注入服务。
- Spot 初始化不要顺手假设 Predict 也已有同级 SDK。

## 动手检查

- `DeepBookClient` 和 `$extend(deepbook(...))` 适用场景有什么差异？
- BalanceManager 配置缺失会影响哪些下单方法？
- Predict raw Move 封装为什么应放在单独服务里？
