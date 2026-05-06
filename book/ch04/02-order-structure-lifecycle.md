# ch04-02 Order 结构和生命周期

[返回本章](README.md)

## 本节目标

- 明确 Order 结构和生命周期 在 `Pool -> Book -> OrderInfo/Order` 撮合链中的职责。
- 能指出本节涉及的订单字段、排序字段、事件或退款字段来自哪个 Move 模块。
- 能用一笔具体订单解释价格优先、时间优先、成交、挂单、取消或查询结果如何产生。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：`Order` 字段、`get_order_id`、`generate_fill`、`locked_balance` 和取消退款。

## 源码定义

`Order` 是真正会留在订单簿里的 maker 状态：

```move
public struct Order has drop, store {
    balance_manager_id: ID,
    order_id: u128,
    client_order_id: u64,
    quantity: u64,
    filled_quantity: u64,
    fee_is_deep: bool,
    order_deep_price: OrderDeepPrice,
    epoch: u64,
    status: u8,
    expire_timestamp: u64,
}
```

字段要按三组理解：

- 归属字段：`balance_manager_id`、`client_order_id`。
- 撮合字段：`order_id`、`quantity`、`filled_quantity`、`status`、`expire_timestamp`。
- 费用字段：`fee_is_deep`、`order_deep_price`、`epoch`。

`Order` 没有 `trader: address` 字段，因为订单归属以 `BalanceManager` 为中心。事件中看到的 trader，是在取消、成交或事件生成时补出来的上下文，不是订单簿长期状态。

## 正文

`order.move` 的 `Order` 是挂在订单簿里的 maker 状态，字段包括：

- `balance_manager_id`：订单归属账户。
- `order_id`：编码后的 u128 订单 ID。
- `client_order_id`：调用方自定义 ID。
- `quantity` 和 `filled_quantity`：原始数量和已成交数量。
- `fee_is_deep`、`order_deep_price`：maker 费用计算所需数据。
- `epoch`：用于历史 maker fee。
- `status`：live、partially_filled、filled、canceled、expired。
- `expire_timestamp`：订单过期时间。

生命周期：

```text
new OrderInfo
  -> match maker orders
  -> if remaining and allowed, to_order
  -> Book injects Order
  -> later generate_fill / modify / cancel / expire
  -> State removes open order and settles balances
```

`Order` 本身不保存 trader address，只保存 BalanceManager ID。事件里的 trader 由 taker 或取消调用者上下文补充。

补充阅读：阅读 Order 结构和生命周期 时，先从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的入口确认交易类型，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消路径。taker 的即时执行状态保存在 `OrderInfo`，maker 的可挂单状态保存在 `Order`，这两个对象不要混在一起看。

边界条件主要来自价格是否穿透、数量是否满足最小 lot、maker 是否过期，以及 taker 执行策略是否允许剩余挂单。手工推演时把每一次 `Fill` 后的 maker 剩余量和 taker 剩余量写出来，最容易发现状态跳转错误。

## 开发要点

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 检查问题

- Order 结构和生命周期 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
