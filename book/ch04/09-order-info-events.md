# ch04-09 OrderInfo 如何生成订单和成交事件

[返回本章](README.md)

## 先看订单现场

读这一节时，不要从函数名开始。先问一笔订单走到“OrderInfo 如何生成订单和成交事件”这个环节时，排序、成交、挂单、取消或退款中哪一个状态会发生变化。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：`Order` 字段、`get_order_id`、`generate_fill`、`locked_balance` 和取消退款。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：`validate_inputs`、`match_maker`、`assert_execution` 和订单生命周期事件。
- [packages/deepbook/sources/book/fill.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/fill.move)：`Fill` 的 maker 方向、成交数量、completed/expired 和结算字段。

## 关键定义

`OrderInfo` 是一次新订单从进入撮合到返回结果的临时对象：

```move
public struct OrderInfo has copy, drop, store {
    pool_id: ID,
    order_id: u128,
    balance_manager_id: ID,
    client_order_id: u64,
    trader: address,
    order_type: u8,
    self_matching_option: u8,
    price: u64,
    is_bid: bool,
    original_quantity: u64,
    executed_quantity: u64,
    cumulative_quote_quantity: u64,
    fills: vector<Fill>,
    fee_is_deep: bool,
    paid_fees: u64,
    maker_fees: u64,
    epoch: u64,
    status: u8,
    market_order: bool,
    fill_limit_reached: bool,
    order_inserted: bool,
    timestamp: u64,
}
```

这组字段把“订单意图”和“执行结果”放在同一个结构里。`original_quantity` 是用户输入，`executed_quantity` 是成交 base 数量，`cumulative_quote_quantity` 是成交 quote 总量，`fills` 保存每个 maker 切片。`order_inserted` 很关键：它告诉你这笔订单是否最终进入订单簿。完全吃单、IOC 剩余取消或 FOK 失败，都不应在前端显示成一个 live maker order。

成交事件也直接来自 `OrderInfo` 中的 fill：

```move
public struct OrderFilled has copy, drop, store {
    pool_id: ID,
    maker_order_id: u128,
    taker_order_id: u128,
    price: u64,
    taker_is_bid: bool,
    taker_fee: u64,
    maker_fee: u64,
    base_quantity: u64,
    quote_quantity: u64,
    maker_balance_manager_id: ID,
    taker_balance_manager_id: ID,
    timestamp: u64,
}
```

## 把订单走一遍

`OrderInfo` 是一次 taker 订单的完整执行记录，也是事件源。它会 emit：

- 自身 `OrderInfo` 事件：包含订单状态、累计成交、fills、费用等。
- `OrderFilled`：每个未过期 fill 对应一条成交事件。
- `OrderPlaced`：剩余数量进入订单簿。
- `OrderFullyFilled`：taker 或 maker 完全成交。
- `OrderExpired` 或 maker cancel 事件：处理过期 maker 或 cancel maker 自成交策略。

事件生成发生在 Pool 的 `place_order_int` 末尾：

```text
order_info.emit_order_info()
order_info.emit_orders_filled(timestamp)
order_info.emit_order_fully_filled_if_filled(timestamp)
```

Indexer 不应只依赖 `OrderFilled` 判断订单最终状态，还需要消费 `OrderInfo`、`OrderPlaced`、`OrderFullyFilled`、`OrderCanceled`、`OrderExpired`。

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

事件阅读时要把 `OrderPlaced`、`OrderFilled`、`OrderFullyFilled`、`OrderCanceled` 和 `OrderExpired` 串成生命周期。取消和过期还会触发锁定资产释放，退款由 `Order` 计算后进入结算路径。

## 交易实现提醒

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 动手检查

- OrderInfo 如何生成订单和成交事件 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
