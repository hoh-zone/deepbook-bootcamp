# ch12-20 本章实战：PredictService

[返回本章](README.md)

## 本节目标

- 完成 `PredictService` 的版本化服务封装。
- 用 raw `tx.moveCall` 绑定 registry、predict_manager、predict 和 oracle 入口。
- 明确当前 Predict 封装是迁移中接口适配，不是稳定主网 SDK。

## 源码关联

- `book/ch12/code/s04-predict-service/`：Predict 服务骨架。
- `packages/predict/sources/registry.move`、`predict_manager.move`、`predict.move`：raw move call target。
- `packages/predict/simulations/src/runtime.ts`：localnet PTB 执行和状态保存参考。

## 正文

`book/ch12/code/s04-predict-service/` 给出 Predict 封装骨架。由于 Predict 处于独立包演进中，示例要求把 package/object IDs、网络和版本显式写入配置，并通过 raw `tx.moveCall` 绑定当前发布的 Move 入口。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 开发要点

- PredictService 初始化缺少 package/object IDs 或 `predictVersionStatus` 时直接失败。
- raw `tx.moveCall` 封装 manager、deposit、mint、redeem、supply、withdraw，并维护 target 白名单。
- dry run 错误映射覆盖 oracle stale、range key、ask bounds、vault exposure、limiter。

## 检查问题

- PredictService 为什么必须版本化，而不能只传 package ID？
- mint 和 redeem 的 PTB 参数顺序分别需要哪些对象？
- 如何向用户解释 oracle、vault 和 manager 三类失败？
