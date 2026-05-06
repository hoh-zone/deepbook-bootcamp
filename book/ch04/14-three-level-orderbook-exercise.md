# ch04-14 三档订单簿手工推演

[返回本章](README.md)

## 本节目标

- 明确 三档订单簿手工推演 在 `Pool -> Book -> OrderInfo/Order` 撮合链中的职责。
- 能指出本节涉及的订单字段、排序字段、事件或退款字段来自哪个 Move 模块。
- 能用一笔具体订单解释价格优先、时间优先、成交、挂单、取消或查询结果如何产生。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：`Order` 字段、`get_order_id`、`generate_fill`、`locked_balance` 和取消退款。
- [packages/deepbook/sources/book/fill.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/fill.move)：`Fill` 的 maker 方向、成交数量、completed/expired 和结算字段。
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)：分页读取 bids/asks 的只读接口，服务前端订单簿和 indexer 对账。

## 正文

假设 asks：

| ask price | quantity | maker |
| --- | --- | --- |
| 100 | 5 | A |
| 101 | 5 | B |
| 103 | 5 | C |

输入 taker bid：`price = 101`、`quantity = 12`、普通限价。

推演：

1. 扫最低 ask 100，成交 5，累计 base=5、quote=500，maker A completed，从 asks 移除。
2. 扫 ask 101，成交 5，累计 base=10、quote=1005，maker B completed，从 asks 移除。
3. 下一个 ask 103 超过 taker price，停止。
4. taker 剩余 2，如果满足 lot/min 约束且不是 IOC/FOK/post-only 终止，则作为 bid price 101 插入 bids。
5. State 计算 taker 买入 10 base 应付 1005 quote 加 taker fee，剩余挂单锁定 2 * 101 quote 加 maker fee。
6. Vault 从 taker BalanceManager 扣 quote/DEEP，把 maker 应收 quote 记入对应 BalanceManager 或 settled balances。

如果同一订单改为 IOC，第 4 步不会插入 bids，剩余 2 被取消。如果改为 FOK，因为没有完全成交 12，会 abort，前两笔成交也回滚。

补充阅读：阅读 三档订单簿手工推演 时，先从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的入口确认交易类型，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消路径。taker 的即时执行状态保存在 `OrderInfo`，maker 的可挂单状态保存在 `Order`，这两个对象不要混在一起看。

边界条件主要来自价格是否穿透、数量是否满足最小 lot、maker 是否过期，以及 taker 执行策略是否允许剩余挂单。手工推演时把每一次 `Fill` 后的 maker 剩余量和 taker 剩余量写出来，最容易发现状态跳转错误。

## 开发要点

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 检查问题

- 三档订单簿手工推演 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
