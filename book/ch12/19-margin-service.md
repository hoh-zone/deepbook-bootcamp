# ch12-19 本章实战：MarginService

[返回本章](README.md)

## 先定封装边界

读这一节时把自己当成 SDK 维护者：封装应该暴露业务意图，隐藏重复参数，同时保留 dry run 和错误定位能力。

## 源码入口

- `book/ch12/code/s03-margin-service/`：Margin 服务骨架。
- `scripts/transactions/marginPrep.ts`、`supplyToMarginPool.ts`、`enableMarginVersion.ts`：管理脚本。
- `packages/deepbook_margin/sources/margin_manager.move`、`margin_pool.move`：用户与池子入口。

## SDK 读法

`book/ch12/code/s03-margin-service/` 给出 Margin admin 和用户接口骨架。管理员函数要求 cap 配置完整；用户函数要求对象状态、oracle freshness 和风险率检查。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- MarginService admin 方法先校验 cap，用户方法先校验 manager、oracle 和风险率。
- borrow/repay/supply/withdraw 分别记录 pool、asset、amount、risk summary。
- 错误解析区分 cap 缺失、版本未启用、oracle stale、风险率不足。

## 动手检查

- MarginService 与 DeepBookService 在风险检查上最大差异是什么？
- 管理员 cap 配置为空时应该在哪一步失败？
- 用户 borrow 后再下单的组合 PTB 需要新增哪些校验？
