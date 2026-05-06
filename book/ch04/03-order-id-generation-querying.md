# ch04-03 订单 ID 的生成、排序和查询

[返回本章](README.md)

## 本节目标

- 明确 订单 ID 的生成、排序和查询 在 `Pool -> Book -> OrderInfo/Order` 撮合链中的职责。
- 能指出本节涉及的订单字段、排序字段、事件或退款字段来自哪个 Move 模块。
- 能用一笔具体订单解释价格优先、时间优先、成交、挂单、取消或查询结果如何产生。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：`Order` 字段、`get_order_id`、`generate_fill`、`locked_balance` 和取消退款。
- [packages/deepbook/sources/helper/utils.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/utils.move)：`encode_order_id`、`decode_order_id` 的 side、price、sequence 编码规则。
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)：分页读取 bids/asks 的只读接口，服务前端订单簿和 indexer 对账。

## 正文

`utils::encode_order_id(is_bid, price, order_id)` 的布局：

- 第 1 bit：bid 为 0，ask 为 1。
- 接下来 63 bits：price。
- 最后 64 bits：同侧递增或递减序号。

bid 不加最高位，ask 加 `1u128 << 127`。`decode_order_id` 返回 `(is_bid, price, order_id)`。

`Book::create_order` 先调用 `get_order_id(order_info.is_bid())`，再编码到 `OrderInfo`。`order_query::iter_orders` 利用这个排序特性分页：bids 从高到低迭代，asks 从低到高迭代。

开发实践：前端展示订单 ID 时应保留 u128 字符串，不要转 JavaScript number。需要解析价格时使用 BigInt 按同样位布局解码。

补充阅读：阅读 订单 ID 的生成、排序和查询 时，先从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的入口确认交易类型，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消路径。taker 的即时执行状态保存在 `OrderInfo`，maker 的可挂单状态保存在 `Order`，这两个对象不要混在一起看。

查询侧要保持 `order_id` 为字符串或 BigInt。[packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move) 返回链上排序后的订单视图，前端聚合深度时只能按 price level 合并数量，不能重新用 JavaScript number 排序 u128。

## 开发要点

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 检查问题

- 订单 ID 的生成、排序和查询 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
