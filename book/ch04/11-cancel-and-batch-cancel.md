# ch04-11 取消单和批量取消路径

[返回本章](README.md)

## 本节目标

- 明确 取消单和批量取消路径 在 `Pool -> Book -> OrderInfo/Order` 撮合链中的职责。
- 能指出本节涉及的订单字段、排序字段、事件或退款字段来自哪个 Move 模块。
- 能用一笔具体订单解释价格优先、时间优先、成交、挂单、取消或查询结果如何产生。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：`Order` 字段、`get_order_id`、`generate_fill`、`locked_balance` 和取消退款。

## 正文

取消入口在 `pool.move`：

- `cancel_order`：必须成功取消指定订单。
- `cancel_orders`：逐个调用 `cancel_order`，任一失败会导致整笔交易失败。
- `cancel_live_order`：如果订单不在该 BalanceManager 的 open orders 中则 no-op。
- `cancel_live_orders`：批量 no-op 版本。
- `cancel_all_orders`：从账户 open orders 取全部订单并取消。

核心链路：

```text
pool::cancel_order
  -> book::cancel_order
  -> assert order.balance_manager_id == balance_manager.id
  -> state::process_cancel
    -> order.set_canceled
    -> order.calculate_cancel_refund
    -> account.remove_order
    -> account.settle
  -> vault::settle_balance_manager
  -> order.emit_order_canceled
```

修改订单 `modify_order` 只能降低数量。它调用 `book::modify_order`，再通过 `state::process_modify` 退回减少部分对应的锁定资产和 maker fee。

补充阅读：阅读 取消单和批量取消路径 时，先从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的入口确认交易类型，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消路径。taker 的即时执行状态保存在 `OrderInfo`，maker 的可挂单状态保存在 `Order`，这两个对象不要混在一起看。

事件阅读时要把 `OrderPlaced`、`OrderFilled`、`OrderFullyFilled`、`OrderCanceled` 和 `OrderExpired` 串成生命周期。取消和过期还会触发锁定资产释放，退款由 `Order` 计算后进入结算路径。

## 开发要点

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 检查问题

- 取消单和批量取消路径 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
