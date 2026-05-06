# ch12-10 撤单、批量撤单和订单查询

[返回本章](README.md)

## 先定封装边界

SDK 小节先看封装边界：好的服务层不是隐藏 Move，而是减少对象、精度、权限和网络配置错误，同时保留 dry run 与错误定位能力。

## 源码入口

- `packages/deepbook/sources/pool.move`：cancel/order query 入口。
- `scripts/transactions/deepbookMarketMaker.ts`：订单服务调用模式。
- `book/ch12/code/s02-deepbook-service/`：批量撤单服务骨架。

## SDK 读法

撤单入口包括 `cancel_order`、`cancel_orders`、`cancel_live_order`、`cancel_live_orders` 和 `cancel_all_orders`。服务层应统一返回撤单 PTB，并在调用前查询订单是否属于当前 `BalanceManager`。订单查询可绑定 `get_order`、`get_orders`、`get_account_order_details`、`account_open_orders`。

批量撤单需要限制单笔 PTB 中的订单数，避免 gas 估算过高或对象读写集合过大。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- 撤单前查询订单归属、pool、orderId 和 BalanceManager。
- 批量撤单限制单笔订单数，并对失败订单返回逐项结果。
- 查询方法与写交易分离，避免只为读订单构造 PTB。

## 动手检查

- cancel order 为什么要先确认订单属于当前 manager？
- 批量撤单过大可能触发哪些 gas 或对象集合问题？
- 订单已成交、已取消、不存在的提示应如何区分？
