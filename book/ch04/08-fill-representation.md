# ch04-08 Fill 如何表示一次成交

[返回本章](README.md)

## 先看订单现场

先把“Fill 如何表示一次成交”放进一笔订单里看。用户看到的是买卖按钮，链上真正发生的是 Pool 接住入口、Book 决定撮合、OrderInfo 带着执行结果回到资金结算。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/fill.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/fill.move)：`Fill` 的 maker 方向、成交数量、completed/expired 和结算字段。

## 关键定义

`Fill` 是一次 taker 和一个 maker 的成交切片：

```move
public struct Fill has copy, drop, store {
    maker_order_id: u128,
    maker_client_order_id: u64,
    execution_price: u64,
    balance_manager_id: ID,
    expired: bool,
    completed: bool,
    original_maker_quantity: u64,
    base_quantity: u64,
    quote_quantity: u64,
    taker_is_bid: bool,
    maker_epoch: u64,
    maker_deep_price: OrderDeepPrice,
    taker_fee: u64,
    taker_fee_is_deep: bool,
    maker_fee: u64,
    maker_fee_is_deep: bool,
}
```

这里的 `balance_manager_id` 是 maker 的 manager，不是 taker。`taker_is_bid` 决定结算方向：taker 买入时 maker 卖出 base、收 quote；taker 卖出时 maker 买入 base、付 quote。`expired` 和 `completed` 不是 UI 状态装饰，它们决定 Book 是否移除 maker order，以及 State 如何释放或结算锁定资产。

## 把订单走一遍

`fill.move` 的 `Fill` 是 taker 与一个 maker order 的匹配结果。字段包括 maker order id、maker client order id、execution price、maker BalanceManager、是否 expired、是否 completed、原始 maker quantity、本次 base/quote 数量、taker side、maker epoch、maker deep price、maker/taker fee。

`Order::generate_fill` 负责创建 Fill：

- 如果 maker 过期或被 `cancel_maker` 策略处理，`expired = true`，base/quote 数量表示要释放的 maker 剩余锁定量。
- 如果正常成交，更新 maker `filled_quantity`，状态变为 partially filled 或 filled。
- `completed` 表示 maker 是否完全成交，Book 后续用它决定是否 remove。

`Fill::get_settled_maker_quantities` 根据 taker side 判断 maker 应收到 base 还是 quote。比如 taker 是 bid，maker 是 ask，正常成交后 maker 收 quote。

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

事件阅读时要把 `OrderPlaced`、`OrderFilled`、`OrderFullyFilled`、`OrderCanceled` 和 `OrderExpired` 串成生命周期。取消和过期还会触发锁定资产释放，退款由 `Order` 计算后进入结算路径。

## 交易实现提醒

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 动手检查

- Fill 如何表示一次成交 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
