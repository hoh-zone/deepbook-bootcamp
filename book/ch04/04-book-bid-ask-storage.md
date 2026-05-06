# ch04-04 Book 如何保存买卖两侧订单

[返回本章](README.md)

## 先看订单现场

先把“Book 如何保存买卖两侧订单”放进一笔订单里看。用户看到的是买卖按钮，链上真正发生的是 Pool 接住入口、Book 决定撮合、OrderInfo 带着执行结果回到资金结算。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：`Order` 字段、`get_order_id`、`generate_fill`、`locked_balance` 和取消退款。

## 关键定义

`Book` 是订单簿真正保存排序状态的结构体。`Pool` 负责权限、资金和版本门禁，`Book` 只关心价格层、订单序号和两侧订单容器。

```move
public struct Book has store {
    tick_size: u64,
    lot_size: u64,
    min_size: u64,
    bids: BigVector<Order>,
    asks: BigVector<Order>,
    next_bid_order_id: u64,
    next_ask_order_id: u64,
}
```

这里最容易误读的是 `next_bid_order_id` 和 `next_ask_order_id`。它们不是前端展示用的自增整数，而是参与 `u128 order_id` 编码的序号来源。bid 和 ask 的排序方向不同，最终都被压进一个可比较的 order id 中，所以应用层必须把 `order_id` 当作链上排序键，而不是普通数据库自增主键。

## 把订单走一遍

`Book` 使用 `BigVector<Order>` 保存 bids 和 asks，而不是普通 vector。它需要支持按 encoded order id 查找、插入、删除、区间扫描和分页。

关键函数：

- `empty(tick_size, lot_size, min_size, ctx)`：初始化两侧 BigVector 和订单序号。
- `book_side_mut(order_id)`：通过解码 order id 判断订单在哪一侧。
- `cancel_order(order_id)`：从对应侧移除订单并返回 `Order`。
- `modify_order(order_id, new_quantity, timestamp)`：降低订单数量，不能增加数量。
- `get_level2_range_and_ticks`：按价格范围聚合 level2 深度。

Book 不维护账户开放订单列表；那部分在 `state/account.move`。所以取消订单时 Pool 必须同时更新 Book 和 State。

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

边界条件主要来自价格是否穿透、数量是否满足最小 lot、maker 是否过期，以及 taker 执行策略是否允许剩余挂单。手工推演时把每一次 `Fill` 后的 maker 剩余量和 taker 剩余量写出来，最容易发现状态跳转错误。

## 交易实现提醒

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 动手检查

- Book 如何保存买卖两侧订单 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
