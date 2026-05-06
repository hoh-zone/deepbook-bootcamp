# ch12-01 SDK 章的边界

[返回本章](README.md)

## 先定封装边界

读这一节时把自己当成 SDK 维护者：封装应该暴露业务意图，隐藏重复参数，同时保留 dry run 和错误定位能力。

## 源码入口

- `scripts/transactions/deepbookMarketMaker.ts`：继承式 `DeepBookClient` 初始化和交易封装。
- `scripts/transactions/createPool.ts`、`prepBalanceManager.ts`：插件式 DeepBook SDK 调用。
- `packages/predict/sources/predict.move`、`predict_manager.move`、`registry.move`：Predict 当前 raw Move 封装边界。

## SDK 读法

SDK 层要做三件事：

1. 把对象 ID、package ID、coin key、pool key、BalanceManager key 从业务代码中抽离。
2. 把一组 Move 调用组合成 PTB，让上层只传业务参数。
3. 在签名前完成模拟、Gas、错误解析和对象状态检查。

DeepBook SDK 有两类常见初始化路径。第一类是 `DeepBookClient` 类实例，适合后端 worker、做市脚本和测试脚本；`scripts/transactions/deepbookMarketMaker.ts` 就是这种写法。第二类是 `SuiGrpcClient().$extend(deepbook(...))` 插件式 client，适合把 `client.deepbook.deepBookAdmin`、`client.deepbook.balanceManager`、`client.deepbook.marginPool` 分层调用；`scripts/transactions/createPool.ts` 和 `prepBalanceManager.ts` 使用这种写法。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- 服务边界统一为配置加载、PTB 构造、dry run、错误解析和钱包签名交接。
- Spot/Margin 优先用 SDK builder，Predict 用 raw Move target 并标注版本状态。
- 不要在 SDK 章引入托管私钥或隐藏对象 ID 的黑盒服务。

## 动手检查

- 本章哪些能力属于 SDK 封装，哪些属于后端或钱包职责？
- Predict 为什么不能直接假设存在 Spot 同形 SDK？
- 一个生产服务初始化最少需要哪些配置字段？
