# ch12-06 查询池子、资产元数据和对象状态

[返回本章](README.md)

## 本节目标

- 查询 pool、资产元数据、BalanceManager、Margin/Predict 对象状态。
- 把链上对象状态、RPC 返回和前端缓存分层。
- 在交易构造前验证对象版本和网络。

## 源码关联

- `packages/deepbook/sources/pool.move`、`balance_manager.move`：Spot 对象状态。
- `packages/deepbook_margin/sources/margin_manager.move`、`margin_pool.move`：Margin 状态。
- `packages/predict/sources/predict.move`、`oracle.move`、`vault/vault.move`：Predict 状态。

## 正文

查询层通常分成两类：

- 链上对象查询：`client.getObject`、`client.multiGetObjects`，用于确认 Pool、BalanceManager、MarginManager、cap 是否存在。
- SDK 或 Move view 查询：池子参数、订单、账户余额、可交易检查。对应源码在 `packages/deepbook/sources/pool.move` 的 `get_order`、`get_orders`、`account_open_orders`、`locked_balance`、`can_place_limit_order`、`can_place_market_order`。

交易前检查要覆盖 pool key、BalanceManager key、订单价格和数量精度、余额、DEEP 支付策略、对象版本是否过旧。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 开发要点

- 查询对象时请求 content、owner、type、previousTransaction 和 version。
- Spot/Margin/Predict 状态分别映射到 pool、manager/pool、oracle/vault/PLP。
- 缓存只做加速，交易构造前重新校验关键共享对象版本。

## 检查问题

- 查询 pool 和查询 Predict vault 需要关注的字段有什么不同？
- 资产 decimals 错误会影响哪些服务方法？
- 缓存对象版本过期时 dry run 可能出现什么错误？
