# ch12-20 本章实战：PredictService

[返回本章](README.md)

## 先定封装边界

`PredictService` 的难点不是把 `tx.moveCall` 包成函数名，而是给一个仍在演进的协议面建立安全边界。服务必须明确 network、package ID、registry、predict object、oracle、quote asset、版本状态和 raw target 白名单；任何一项模糊，都可能让前端构造出能签名但不能解释的交易。

## 源码入口

- `book/ch12/code/s04-predict-service/`：Predict 服务骨架。
- `packages/predict/sources/registry.move`、`predict_manager.move`、`predict.move`：raw move call target。
- `packages/predict/simulations/src/runtime.ts`：localnet PTB 执行和状态保存参考。

## SDK 读法

`book/ch12/code/s04-predict-service/` 给出 Predict 封装骨架。由于 Predict 处于独立包演进中，示例要求把 package/object IDs、网络和版本显式写入配置，并通过 raw `tx.moveCall` 绑定当前发布的 Move 入口。这里故意不把 Predict 包成和 Spot 完全同形的 SDK：Spot 的对象和公共 SDK 更稳定，Predict 的 Testnet 集成面仍需要更强的版本标注。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

一个合格的 `PredictService` 返回值应该能被日志系统和客服系统使用：输入对象、type arguments、range key、oracle freshness、dry run status、Move abort 和用户可读提示都要保留下来。否则用户只会看到“交易失败”，开发者也无法判断是 oracle stale、vault exposure、range key 还是 manager 余额问题。

## 封装判断

- PredictService 初始化缺少 package/object IDs 或 `predictVersionStatus` 时直接失败。
- raw `tx.moveCall` 封装 manager、deposit、mint、redeem、supply、withdraw，并维护 target 白名单。
- dry run 错误映射覆盖 oracle stale、range key、ask bounds、vault exposure、limiter。
- 用户交易只返回 PTB/bytes 给钱包签名；服务端不托管私钥。

## 动手检查

- PredictService 为什么必须版本化，而不能只传 package ID？
- mint 和 redeem 的 PTB 参数顺序分别需要哪些对象？
- 如何向用户解释 oracle、vault 和 manager 三类失败？
