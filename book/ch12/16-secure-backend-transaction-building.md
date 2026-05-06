# ch12-16 后端安全构造交易

[返回本章](README.md)

## 本节目标

- 设计后端安全构造交易的 API 边界。
- 后端保存请求、报价快照和 dry run 摘要，不保存私钥。
- 对 Predict 交易强制校验版本状态和对象 ID。

## 源码关联

- `book/ch12/code/s05-wallet-flow/`、`s06-dry-run-helper/`：安全构造和错误解析。
- `scripts/config/constants.ts`：对象 ID 白名单。
- `packages/predict/sources/predict.move`：Predict 交易 target 白名单。

## 正文

后端服务可以构造交易，但不应替用户签名。推荐接口是：

```ts
POST /deepbook/orders/limit/build
{
  "sender": "0x...",
  "poolKey": "SUI_DBUSDC",
  "balanceManagerKey": "USER_MANAGER",
  "price": 1.23,
  "quantity": 10,
  "isBid": true
}
```

返回交易 bytes 和 dry run 摘要。后端只保存请求、报价快照和 dry run 结果，不保存私钥、不代签用户订单。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 开发要点

- 后端 API 校验 sender、pool/market key、manager key、金额、滑点和版本状态。
- 响应包含 bytes、dry run 摘要、报价快照、过期时间和错误码。
- 服务端日志保存 digest/requestId，不保存签名材料。

## 检查问题

- 构造交易 API 哪些参数必须由后端二次校验？
- 报价快照和 dry run 摘要为什么要一起返回？
- Predict 交易缺少迁移状态时后端应如何拒绝？
