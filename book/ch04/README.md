# ch04 Pool、Book 与撮合源码精读

## 本章目标

- 读懂 `Pool` 下单入口、`Book` 撮合循环、`Order` maker 状态、`Fill` 成交结果和 `OrderInfo` taker 生命周期。
- 能解释 bid、ask、price level、queue priority 如何编码到 `order_id` 和 `BigVector<Order>` 中。
- 能推演限价单、市价单、IOC、FOK、post-only、自成交策略、取消和修改订单的状态变化。
- 能把链上事件和源码中的状态字段对应起来，服务前端订单查询和 indexer 落库。

## 本章学习阶梯

- L2 先用三档订单簿手工推演 bid、ask、price level 和 fill。
- L3 再读订单 ID、撮合循环、剩余挂单和事件生成。
- L4 能解释 post-only、IOC、FOK、自成交策略的失败边界。
- L5 能把撮合结果还原成前端订单状态和 Indexer 数据。

## 源码地图

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：`place_limit_order`、`place_market_order`、`modify_order`、`cancel_order`、`cancel_live_order` 等外部入口。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`Book` 字段、`create_order`、`match_against_book`、`inject_limit_order`、`cancel_order`、`modify_order`。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：`Order` 字段、`generate_fill`、`calculate_cancel_refund`、`locked_balance`。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：`OrderInfo` 字段、`validate_inputs`、`assert_execution`、`match_maker`、事件生成。
- [packages/deepbook/sources/book/fill.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/fill.move)：`Fill` 字段和 maker 结算方向。
- [packages/deepbook/sources/helper/utils.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/utils.move)：`encode_order_id`、`decode_order_id`。
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)：分页读取 bids/asks 的查询接口。

## 小节目录

- [01 bid、ask、price level、queue priority](01-bids-asks-price-levels-queue-priority.md)
- [02 Order 结构和生命周期](02-order-structure-lifecycle.md)
- [03 订单 ID 的生成、排序和查询](03-order-id-generation-querying.md)
- [04 Book 如何保存买卖两侧订单](04-book-bid-ask-storage.md)
- [05 限价单进入撮合引擎的完整路径](05-limit-order-matching-path.md)
- [06 市价单和吃单资金检查](06-market-order-fund-checks.md)
- [07 post-only、IOC、FOK、自成交策略](07-post-only-ioc-fok-self-match.md)
- [08 Fill 如何表示一次成交](08-fill-representation.md)
- [09 OrderInfo 如何生成订单和成交事件](09-order-info-events.md)
- [10 部分成交、完全成交、剩余挂单](10-partial-full-fill-resting-order.md)
- [11 取消单和批量取消路径](11-cancel-and-batch-cancel.md)
- [12 order query 如何服务前端查询](12-order-query-for-frontends.md)
- [13 撮合边界条件阅读方法](13-matching-edge-cases.md)
- [14 三档订单簿手工推演](14-three-level-orderbook-exercise.md)

## 本章代码

- `code/s01-orderbook-simulator/`：用 TypeScript 实现最小订单簿模拟器。
- `code/s02-match-trace/`：输入订单列表，输出逐步撮合日志。
- `code/s03-order-query/`：查询链上订单并与本地模拟排序规则对比。

## Move 高阶穿插点

- 撮合源码要按“输入校验、订单构造、撮合循环、剩余处理、事件输出、资金结算”六段读。
- 订单簿里的整数不要急着换成小数显示；先保留链上原始单位，最后一层 UI 再格式化。
- `OrderInfo` 和 `Order` 要分开理解：前者描述本次交易结果，后者描述可留在簿上的状态。

## 常见错误

- 错误：把市价单当成没有价格。源码中市价买单使用 max price，市价卖单使用 min price，并强制 IOC。
- 错误：用 JavaScript number 保存 u128 order id。必须使用字符串或 BigInt。
- 错误：认为部分成交普通限价单的状态一定是 live。`OrderInfo` 可是 partially_filled，同时剩余部分以 `Order` 进入 Book。
- 错误：取消订单只删 Book。实际还要更新 account open orders、计算 refund、通过 Vault 结算。
- 错误：只监听 `OrderFilled`。订单生命周期还需要 `OrderInfo`、`OrderPlaced`、`OrderFullyFilled`、`OrderCanceled`、`OrderExpired`。

## 本章检查清单

- [ ] 能解释 encoded order id 的 bit 布局。
- [ ] 能说明 bid/ask 分别从哪一侧、哪个价格方向开始撮合。
- [ ] 能写出限价单进入 Book 的完整函数链。
- [ ] 能解释 Fill 中 expired/completed 的不同含义。
- [ ] 能推演 IOC、FOK、post-only 和自成交策略的行为。
- [ ] 能说明取消订单如何退还锁定资产。

## 进阶练习

- 练习 1：用 BigInt 写一个 `decodeOrderId`，输入链上 order id 后输出 side、price、sequence。
- 练习 2：构造一个 maker 过期场景，解释为什么 `Fill.expired = true` 但它仍会参与移除订单和释放锁定资产。
- 练习 3：设计一个包含 4 个 maker 的订单簿，分别推演普通限价、IOC 和 FOK taker 的最终状态。

