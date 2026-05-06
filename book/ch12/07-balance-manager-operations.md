# ch12-07 BalanceManager 操作

[返回本章](README.md)

## 本节目标

- 封装 BalanceManager 创建、deposit、withdraw 和 cap 操作。
- 说明 Spot BalanceManager 与 PredictManager 内部复用模型的差异。
- 为用户交易返回 PTB/bytes 而不是代签。

## 源码关联

- `scripts/transactions/prepBalanceManager.ts`：创建和分享 BalanceManager。
- `packages/deepbook/sources/balance_manager.move`：deposit、withdraw、cap。
- `packages/predict/sources/predict_manager.move`：PredictManager 内部复用 BalanceManager。

## 正文

`BalanceManager` 是 DeepBook 交易资金账户。源码在 `packages/deepbook/sources/balance_manager.move`，核心函数包括 `new`、`deposit`、`withdraw`、`mint_trade_cap`、`generate_proof_as_owner`。SDK 封装至少提供：

- 创建并 share BalanceManager。
- deposit 指定 coin。
- withdraw 指定 coin 和数量。
- 查询 BalanceManager 中的 coin 余额。
- 为子账户或策略进程发放 trade cap、deposit cap、withdraw cap。

`scripts/transactions/prepBalanceManager.ts` 展示了 `balanceManagers` 映射和 `createAndShareBalanceManager` 调用方式。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 开发要点

- BalanceManager 创建、deposit、withdraw 都返回 PTB，不直接签名。
- trade proof/cap 的获取和使用封装在服务层，不暴露给 UI 乱传。
- 说明 PredictManager 是另一层用户对象，不能直接拿 Spot BalanceManager 当 Predict manager。

## 检查问题

- 用户下单前为什么通常需要 BalanceManager？
- cap 或 manager owner 错误会在哪个阶段暴露？
- PredictManager 与 Spot BalanceManager 的关系和差异是什么？
