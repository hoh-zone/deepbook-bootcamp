# ch04-13 撮合边界条件阅读方法

[返回本章](README.md)

## 先看订单现场

读这一节时，不要从函数名开始。先问一笔订单走到“撮合边界条件阅读方法”这个环节时，排序、成交、挂单、取消或退款中哪一个状态会发生变化。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：`validate_inputs`、`match_maker`、`assert_execution` 和订单生命周期事件。

## 把订单走一遍

读撮合边界时优先从 abort code 和状态判断入手：

- `order_info::validate_inputs`：价格范围、tick size、min size、lot size、过期时间、订单类型。
- `order_info::assert_execution`：post-only、FOK、IOC。
- `order_info::match_maker`：价格是否 crossing、自成交策略。
- `book::match_against_book`：`max_fills` 限制和 maker 移除。
- `order::modify`：新数量必须大于已成交且小于原数量。
- `order::calculate_cancel_refund`：取消或修改时退还未成交数量和 maker fee。

测试设计应覆盖：空簿市价单、刚好 min size、非 lot multiple、post-only crossing、FOK 部分成交、IOC 剩余取消、maker 过期、自成交 cancel taker、cancel maker、达到 max fills。

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

边界条件主要来自价格是否穿透、数量是否满足最小 lot、maker 是否过期，以及 taker 执行策略是否允许剩余挂单。手工推演时把每一次 `Fill` 后的 maker 剩余量和 taker 剩余量写出来，最容易发现状态跳转错误。

## 交易实现提醒

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 动手检查

- 撮合边界条件阅读方法 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
