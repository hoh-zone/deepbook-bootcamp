# ch01-03 CLOB、AMM、RFQ 和聚合器

[返回本章](README.md)

## 本节目标

- 判断 DeepBook 在聚合交易路径中的协议角色。
- 能沿“CLOB、AMM、RFQ 和聚合器”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)
- [crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server)
- [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

CLOB 路径：钱包签名 PTB，传入 `Pool<BaseAsset, QuoteAsset>`、`BalanceManager`、`TradeProof`、价格、数量和方向，链上撮合后更新订单簿和账户状态，并发出事件。

AMM 路径：钱包签名交易，传入池子和输入资产，合约按曲线计算输出。交易主要关注滑点、储备量和手续费。

RFQ 路径：报价方离线或链下给出价格，交易者接受报价后链上结算。核心风险在报价有效期、签名校验和对手方流动性。

聚合器路径：聚合器把多个 CLOB、AMM 或 RFQ 拆成一笔或多笔路由。应用开发者仍要识别底层协议类型，因为错误处理、事件解析和资金结算完全不同。

DeepBookV3 的交易入口在 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)；撮合核心在 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)；订单事件在 [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move) 和 [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)。

## 阅读补充

聚合器接入 DeepBook 时，通常不会只关心“能不能换币”，还要关心可成交深度、滑点保护和交易失败后的回退路径。CLOB 给出的报价依赖当前订单簿状态；RFQ 依赖做市商报价承诺；AMM 依赖曲线和储备，因此同一个前端路由需要把三类风险拆开处理。

源码阅读可以先看 `order_query.move` 和 server/indexer 的查询边界，再回到 `pool.move` 的交易入口。这样能分清“报价读模型”与“最终执行交易”分别来自哪里。

## 开发要点

- 聚合器报价要带上时间、对象版本或 digest 上下文，避免展示过期深度。
- 执行前仍要在交易参数里设置可接受价格或数量边界，不能信任链下报价恒定有效。
- 把 DeepBook 事件纳入成交回放，方便解释路由结果和用户实际到账。

## 检查问题

- CLOB、AMM、RFQ 的报价可信来源分别是什么？
- 聚合器调用 DeepBook 时，哪部分是链下查询，哪部分是链上执行？
- 订单簿深度变化会怎样影响路由失败率？
