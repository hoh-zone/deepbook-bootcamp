# ch12-11 swap/route 封装

[返回本章](README.md)

## 先定封装边界

读这一节时把自己当成 SDK 维护者：封装应该暴露业务意图，隐藏重复参数，同时保留 dry run 和错误定位能力。

## 源码入口

- `packages/deepbook/sources/pool.move`：swap 与 flashloan 相关入口。
- `scripts/transactions/deepbookMarketMaker.ts`：borrow/swap/return PTB 示例。
- `book/ch12/code/s02-deepbook-service/`：route 服务扩展位置。

## SDK 读法

单池 swap 可以直接封装 SDK 的 `deepBook.swapExactQuoteForBase` 或 `deepBook.swapExactBaseForQuote`。多池 route 应拆成：

1. 报价阶段：读取每个池子的 depth、tick、fee、滑点。
2. 构造阶段：按路径串联 PTB，前一跳输出 coin 进入后一跳。
3. 校验阶段：dry run 后检查 `balanceChanges` 和 `objectChanges`。

`scripts/transactions/deepbookMarketMaker.ts` 的闪电贷示例展示了一个 PTB 中串联 borrow、swap、transfer、return 的方式。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- route 报价阶段和 PTB 构造阶段分离，报价快照写入 dry run 摘要。
- 多跳 PTB 用前一跳输出 coin 连接后一跳，并设置最终最小输出。
- 闪电贷组合必须保证 borrow 和 return 在同一 PTB 内闭合。

## 动手检查

- 单池 swap 和多池 route 的错误面有什么差异？
- dry run 后应检查哪些 balanceChanges？
- 闪电贷 PTB 如果中途失败，为什么不会留下未归还状态？
