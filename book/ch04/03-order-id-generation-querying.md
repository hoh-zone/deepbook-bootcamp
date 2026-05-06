# ch04-03 订单 ID 的生成、排序和查询

[返回本章](README.md)

## 先看订单现场

读这一节时，不要从函数名开始。先问一笔订单走到“订单 ID 的生成、排序和查询”这个环节时，排序、成交、挂单、取消或退款中哪一个状态会发生变化。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：`Order` 字段、`get_order_id`、`generate_fill`、`locked_balance` 和取消退款。
- [packages/deepbook/sources/helper/utils.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/utils.move)：`encode_order_id`、`decode_order_id` 的 side、price、sequence 编码规则。
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)：分页读取 bids/asks 的只读接口，服务前端订单簿和 indexer 对账。

## 把订单走一遍

`utils::encode_order_id(is_bid, price, order_id)` 的布局：

- 第 1 bit：bid 为 0，ask 为 1。
- 接下来 63 bits：price。
- 最后 64 bits：同侧递增或递减序号。

bid 不加最高位，ask 加 `1u128 << 127`。`decode_order_id` 返回 `(is_bid, price, order_id)`。

`Book::create_order` 先调用 `get_order_id(order_info.is_bid())`，再编码到 `OrderInfo`。`order_query::iter_orders` 利用这个排序特性分页：bids 从高到低迭代，asks 从低到高迭代。

开发实践：前端展示订单 ID 时应保留 u128 字符串，不要转 JavaScript number。需要解析价格时使用 BigInt 按同样位布局解码。

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

查询侧要保持 `order_id` 为字符串或 BigInt。[packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move) 返回链上排序后的订单视图，前端聚合深度时只能按 price level 合并数量，不能重新用 JavaScript number 排序 u128。

## 交易实现提醒

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 动手检查

- 订单 ID 的生成、排序和查询 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
