# ch12-09 市价单封装

[返回本章](README.md)

## 本节目标

- 封装市价单和 swap 类订单。
- 明确 base/quote 数量语义、部分成交和滑点保护。
- 用 dry run 校验 balanceChanges 和最小成交。

## 源码关联

- `packages/deepbook/sources/pool.move`：`place_market_order`、swap 入口。
- `scripts/transactions/deepbookMarketMaker.ts`：服务封装参考。
- `scripts/utils/utils.ts`：dry run 后检查 balanceChanges。

## 正文

市价单对应 `place_market_order`，应用层通常以“花多少 quote 买 base”或“卖多少 base 换 quote”的方式表达。服务层要明确：

- `isBid` 的含义。
- `quantity` 是 base 数量还是 quote 数量。
- 是否允许部分成交。
- 最小成交数量或滑点保护。

对 swap 类交易，SDK 还提供 `swapExactQuoteForBase`、`swapExactBaseForQuote` 等更接近前端 swap 的方法，底层对应 `pool.move` 的 swap 入口。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 开发要点

- 市价单接口明确 exact base、exact quote 或 swap exact in/out。
- 设置最大滑点或最小输出，并在 dry run 后校验 balanceChanges。
- 部分成交策略必须由调用方显式选择。

## 检查问题

- 市价买入时 quantity 表示 base 还是 quote？
- 没有最小输出保护会产生什么用户风险？
- 市价单和 swap 封装何时应使用不同方法名？
