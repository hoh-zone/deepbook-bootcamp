# ch10-04 `predict_manager.move` 的用户账户和头寸管理

[返回本章](README.md)

## 本节目标

- 理解 `PredictManager` 为什么是用户级共享对象，而不是普通前端本地账户。
- 说明内部 DeepBook `BalanceManager`、deposit/withdraw cap 和 positions table 如何协作。
- 设计应用加载或创建 manager 时的对象 ID、owner 和版本检查。

## 源码关联

- `packages/predict/sources/predict_manager.move`：`PredictManager` 字段、deposit、withdraw、position 增减。
- `packages/predict/sources/registry.move`：创建和分享 manager 的入口与派生 key。
- `packages/deepbook/sources/balance_manager.move`：Predict 内部复用的余额模型。

## 正文

`PredictManager` 是每个用户的共享对象，字段包括 owner、DeepBook `BalanceManager`、`DepositCap`、`WithdrawCap` 和 `Table<RangeKey, u64>` positions。`registry.move::create_and_share_manager` 使用 `PredictManagerKey(address, 0)` 派生对象 ID，v1 默认每个地址一个 manager。

用户资金先通过 `deposit<T>(manager, coin, ctx)` 进入内部 `BalanceManager`，交易时 `predict.move` 用包内权限调用 `manager.withdraw<Quote>(cost, ctx)`。头寸只记录 range quantity，不记录成本价；应用层要从事件、交易历史或未来 Indexer 里恢复成本和 PnL。

服务层通常提供 `ensurePredictManager(owner)`：先按 `PredictManagerKey(address, 0)` 查询对象，不存在再构造创建 PTB。创建之后，后续 deposit、mint、redeem 都使用同一个 manager 对象，避免用户在多个 manager 间分散余额和头寸。

因为 positions table 只保存 quantity，成本价、fee、mint 时间和 realized PnL 需要从交易事件或未来 Indexer 读模型恢复。当前迁移状态下，应用可以本地记录提交摘要和 digest，但不能把未完成的 Predict Indexer 当成唯一账本。

## 开发要点

- manager 创建、deposit、mint 应拆成可组合 PTB，允许前端一次签名完成初始化。
- 任何交易都校验 `ctx.sender()` 与 manager owner 一致，避免代签或错误对象。
- PnL 展示明确区分链上 position quantity 和链下成本记录。

## 检查问题

- 为什么 manager ID 派生要包含 owner，而 market key 不包含 owner？
- 如果用户换钱包或误选别人的 manager，Move 层会在哪里拒绝？
- 没有稳定 Indexer 时，应用如何保存和恢复用户成本信息？
