# ch12-18 本章实战：DeepBookService

[返回本章](README.md)

## 先定封装边界

SDK 小节先看封装边界：好的服务层不是隐藏 Move，而是减少对象、精度、权限和网络配置错误，同时保留 dry run 与错误定位能力。

## 源码入口

- `book/ch12/code/s02-deepbook-service/`：Spot 服务骨架。
- `scripts/transactions/deepbookMarketMaker.ts`、`prepBalanceManager.ts`：SDK 调用参考。
- `packages/deepbook/sources/pool.move`、`balance_manager.move`：Move 入口。

## SDK 读法

`book/ch12/code/s02-deepbook-service/` 给出 Spot 服务封装骨架：初始化 SDK、创建 BalanceManager、deposit、限价单、市价单、撤单、swap 和查询。它的核心原则是业务层只看到 `poolKey` 和金额，不接触 Move object 顺序。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- DeepBookService 对外只暴露 poolKey、managerKey、side、price、quantity 等业务参数。
- 内部负责 SDK 初始化、对象解析、PTB 构造、dry run 和错误归类。
- Spot 服务不处理 Predict oracle/vault 逻辑，只在共享配置层保持一致格式。

## 动手检查

- DeepBookService 哪些方法返回 PTB，哪些方法返回查询结果？
- 如何隐藏 Move object 顺序但保留错误可定位性？
- BalanceManager 缺失时服务应自动创建还是返回待签 PTB？
