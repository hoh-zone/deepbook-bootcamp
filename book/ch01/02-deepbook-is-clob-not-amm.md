# ch01-02 DeepBook 是 CLOB，不是 AMM

[返回本章](README.md)

## 先看问题

先把“DeepBook 是 CLOB，不是 AMM”放到读者路径里看：你不是在背一个协议名，而是在建立一套判断 DeepBook 能做什么、不能做什么的坐标。读这一节时，重点看产品边界如何落到对象、交易和数据系统上。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 建立直觉

CLOB 的核心是挂单簿。买单和卖单按价格、时间或订单 ID 排序，taker 的新订单进入撮合路径，maker 的剩余订单可能留在簿上。AMM 的核心是资金池曲线，交易者和池子交换资产，价格由曲线和储备量推导。

DeepBookV3 的入口说明在 [README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/README.md) 中直接称其为 decentralized central limit order book。合约入口位于 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)，其中 `place_limit_order` 和 `place_market_order` 都返回 `OrderInfo`，说明交易结果不是简单的 swap 数值，而是订单生命周期的一部分。

开发时不要用 AMM 的“输入 coin，输出 coin，路径结束”来理解所有 DeepBook 操作。DeepBookV3 支持直接 swap，但常规交易会经过 `BalanceManager`、`TradeProof`、`Pool`、`Book`、`State`、`Vault`，并生成订单和成交事件。

## 阅读补充

阅读 CLOB 路径时，从 `pool.move` 的下单入口开始，看它如何把订单参数转成 `OrderInfo`，再交给 `book.move` 与对手盘匹配。AMM 的核心状态通常是池内储备和曲线公式；DeepBook 的核心状态还包括价格档位、订单 ID、maker/taker 账户影响和事件流。

直接 swap 只是 DeepBook 暴露的一种使用方式，不代表底层退化成 AMM。开发者在前端展示成交结果时，仍要考虑订单簿深度、部分成交、剩余挂单和订单事件。

## 落地判断

- 不要用 AMM 的 reserve ratio 推断 DeepBook 报价，必须读取订单簿或查询服务给出的深度。
- 构造限价单时写清 `is_bid`、价格、数量、订单类型和 `BalanceManager` 授权。
- 调试成交差异时同时看 `OrderInfo`、fill 事件和 Vault 净额结算。

## 读完以后问自己

- CLOB 的价格来自哪里，AMM 的价格又来自哪里？
- 一笔限价单为什么可能产生部分成交和剩余挂单？
- DeepBook 的 swap 入口为什么仍需要理解订单簿？
