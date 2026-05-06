# ch12-15 钱包签名、提交、确认和错误解析

[返回本章](README.md)

## 本节目标

- 设计钱包签名、提交、确认和错误解析流程。
- 服务端只返回 Transaction/bytes，前端调用钱包签名。
- 解析 effects、events、objectChanges 和 balanceChanges 定位失败原因。

## 源码关联

- `book/ch12/code/s05-wallet-flow/`：钱包流程骨架。
- `scripts/utils/utils.ts`：提交前 dry run 和交易 bytes。
- `packages/predict/sources/helper/constants.move`：Predict abort/常量映射参考。

## 正文

前端钱包流程应只接收 `Transaction` 或 bytes：

```ts
const result = await wallet.signAndExecuteTransaction({
  transaction: tx,
  options: { showEffects: true, showObjectChanges: true, showBalanceChanges: true },
});
```

服务层不要要求用户上传私钥。确认阶段要读取 digest，再用 `client.waitForTransaction` 或轮询交易详情。失败时优先解析：

- `effects.status.error`：Move abort、对象版本、余额、gas。
- `events`：是否已经发出部分业务事件。
- `objectChanges`：共享对象是否被写入。
- `balanceChanges`：资产变化是否符合预期。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 开发要点

- 前端只接收 Transaction/bytes 并调用钱包签名提交。
- 确认阶段读取 digest，再查询 effects、events、objectChanges、balanceChanges。
- 错误解析优先展示可操作原因：余额、对象版本、权限、oracle stale、gas。

## 检查问题

- 为什么后端不应要求用户上传私钥？
- 交易失败但有 objectChanges 时应如何处理展示？
- Predict mint 失败应重点解析哪些模块错误？
