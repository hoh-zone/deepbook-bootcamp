# ch04-10 部分成交、完全成交、剩余挂单

[返回本章](README.md)

## 先看订单现场

先把“部分成交、完全成交、剩余挂单”放进一笔订单里看。用户看到的是买卖按钮，链上真正发生的是 Pool 接住入口、Book 决定撮合、OrderInfo 带着执行结果回到资金结算。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：`validate_inputs`、`match_maker`、`assert_execution` 和订单生命周期事件。
- [packages/deepbook/sources/book/fill.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/fill.move)：`Fill` 的 maker 方向、成交数量、completed/expired 和结算字段。

## 把订单走一遍

`order_info::match_maker` 每撮合一个 maker：

- 调用 `maker.generate_fill(...)`。
- 如果 fill 未过期，则增加 taker `executed_quantity` 和 `cumulative_quote_quantity`。
- taker 状态先设为 partially filled；如果 remaining 为 0，则设为 filled。

`book::match_against_book` 撮合后会移除 expired 或 completed maker。然后 `assert_execution` 判断 taker 是否终止：

- 完全成交：状态 filled，不挂单。
- IOC：剩余取消，不挂单。
- fill limit reached：不继续扫描，函数返回 true，不挂单。
- 普通限价单仍有剩余：`inject_limit_order`，状态 live/partially filled，`order_inserted = true`。

状态变化表：

| 场景 | taker executed | taker status | maker status | 是否插入 taker |
| --- | --- | --- | --- | --- |
| 完全成交 | original_quantity | filled | filled 或 partially_filled | 否 |
| 部分成交普通限价 | 小于 original | partially_filled | filled/partially_filled | 是 |
| IOC 部分成交 | 小于 original | canceled | filled/partially_filled | 否 |
| post-only crossing | 大于 0 前 abort | abort | 回滚 | 否 |
| FOK 不完全成交 | 小于 original 前 abort | abort | 回滚 | 否 |

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

事件阅读时要把 `OrderPlaced`、`OrderFilled`、`OrderFullyFilled`、`OrderCanceled` 和 `OrderExpired` 串成生命周期。取消和过期还会触发锁定资产释放，退款由 `Order` 计算后进入结算路径。

## 交易实现提醒

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 动手检查

- 部分成交、完全成交、剩余挂单 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
