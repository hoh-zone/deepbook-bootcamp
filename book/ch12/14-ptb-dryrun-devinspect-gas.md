# ch12-14 PTB、dry run、dev inspect 和 Gas

[返回本章](README.md)

## 先定封装边界

SDK 小节先看封装边界：好的服务层不是隐藏 Move，而是减少对象、精度、权限和网络配置错误，同时保留 dry run 与错误定位能力。

## 源码入口

- `scripts/utils/utils.ts`：`dryRunTransactionBlock`、gas payment、multisig bytes。
- `packages/deepbook/sources/pool.move`、`packages/deepbook_margin/sources/*`、`packages/predict/sources/*`：错误定位的 Move target。
- `book/ch12/code/s06-dry-run-helper/`：错误解析工具骨架。

## SDK 读法

PTB 是 SDK 服务层的基本返回值。后端构造后应：

1. 设置 sender。
2. 可选设置 gas payment。
3. `tx.build({ client })`。
4. `client.dryRunTransactionBlock({ transactionBlock })`。
5. 检查 `effects.status.status`。

`scripts/utils/utils.ts` 中的 `prepareMultisigTx` 会设置 gas price、expiration、gas payment，并在写入 multisig bytes 前执行 dry run。这是管理员交易的标准做法。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- 每个写交易执行 build、dry run、status 检查、错误解析，再交给签名流程。
- 管理员 multisig bytes 固定 gas price、expiration 和 gas payment。
- 错误码按 package/module/function 归类，覆盖 Spot、Margin、Predict。

## 动手检查

- dry run 和 dev inspect 在 SDK 服务里分别解决什么问题？
- Gas payment 固定对管理员 multisig 有什么价值？
- Move abort 如何映射到具体入口和用户提示？
