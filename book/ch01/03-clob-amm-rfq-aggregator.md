# ch01-03 CLOB、AMM、RFQ 和聚合器

[返回本章](README.md)

## 先看问题

这一节不急着进源码。先用“CLOB、AMM、RFQ 和聚合器”回答一个更基础的问题：如果要构建真实交易应用，哪些概念必须先讲清楚，哪些细节可以等到后面再拆。

## 本节坐标

本节先把源码范围缩到最少。读的时候只抓产品边界、对象名称、函数入口和事件线索；后面进入交易路径时，再回到这些入口核对细节。

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)
- [crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server)
- [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)

这一组入口用来校准产品边界：先看它们暴露哪些对象和动作，再判断本节概念最终会落到哪条交易或数据路径。

## 建立直觉

CLOB 路径：钱包签名 PTB，传入 `Pool<BaseAsset, QuoteAsset>`、`BalanceManager`、`TradeProof`、价格、数量和方向，链上撮合后更新订单簿和账户状态，并发出事件。

AMM 路径：钱包签名交易，传入池子和输入资产，合约按曲线计算输出。交易主要关注滑点、储备量和手续费。

RFQ 路径：报价方离线或链下给出价格，交易者接受报价后链上结算。核心风险在报价有效期、签名校验和对手方流动性。

聚合器路径：聚合器把多个 CLOB、AMM 或 RFQ 拆成一笔或多笔路由。应用开发者仍要识别底层协议类型，因为错误处理、事件解析和资金结算完全不同。

DeepBookV3 的交易入口在 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)；撮合核心在 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)；订单事件在 [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move) 和 [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)。

## 阅读补充

聚合器接入 DeepBook 时，通常不会只关心“能不能换币”，还要关心可成交深度、滑点保护和交易失败后的回退路径。CLOB 给出的报价依赖当前订单簿状态；RFQ 依赖做市商报价承诺；AMM 依赖曲线和储备，因此同一个前端路由需要把三类风险拆开处理。

源码阅读可以先看 `order_query.move` 和 server/indexer 的查询边界，再回到 `pool.move` 的交易入口。这样能分清“报价读模型”与“最终执行交易”分别来自哪里。

## 落地判断

- 聚合器报价要带上时间、对象版本或 digest 上下文，避免展示过期深度。
- 执行前仍要在交易参数里设置可接受价格或数量边界，不能信任链下报价恒定有效。
- 把 DeepBook 事件纳入成交回放，方便解释路由结果和用户实际到账。

## 读完以后问自己

- CLOB、AMM、RFQ 的报价可信来源分别是什么？
- 聚合器调用 DeepBook 时，哪部分是链下查询，哪部分是链上执行？
- 订单簿深度变化会怎样影响路由失败率？
