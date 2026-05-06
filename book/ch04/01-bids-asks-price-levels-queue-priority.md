# ch04-01 bid、ask、price level、queue priority

[返回本章](README.md)

## 先看订单现场

读这一节时，不要从函数名开始。先问一笔订单走到“bid、ask、price level、queue priority”这个环节时，排序、成交、挂单、取消或退款中哪一个状态会发生变化。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/helper/utils.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/utils.move)：`encode_order_id`、`decode_order_id` 的 side、price、sequence 编码规则。

## 把订单走一遍

bid 是买 base、付 quote；ask 是卖 base、收 quote。DeepBook 的订单数量 `quantity` 始终按 base asset 计量，价格表示每单位 base 对应多少 quote。

`Book` 内有两侧：

```move
bids: BigVector<Order>,
asks: BigVector<Order>,
next_bid_order_id: u64,
next_ask_order_id: u64,
```

`book::match_against_book` 对买单从 asks 的最低价开始扫，对卖单从 bids 的最高价开始扫。这就是 CLOB 的价格优先。相同价格下，`get_order_id` 让 bid 的序号递减、ask 的序号递增，再由 `utils::encode_order_id` 编入低 64 位，形成可排序 key。队列优先级因此由编码后的 `order_id` 和 `BigVector` 顺序共同决定。

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

查询侧要保持 `order_id` 为字符串或 BigInt。[packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move) 返回链上排序后的订单视图，前端聚合深度时只能按 price level 合并数量，不能重新用 JavaScript number 排序 u128。

## 交易实现提醒

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 动手检查

- bid、ask、price level、queue priority 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
