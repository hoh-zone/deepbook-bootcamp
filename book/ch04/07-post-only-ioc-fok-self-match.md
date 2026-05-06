# ch04-07 post-only、IOC、FOK、自成交策略

[返回本章](README.md)

## 本节目标

- 明确 post-only、IOC、FOK、自成交策略 在 `Pool -> Book -> OrderInfo/Order` 撮合链中的职责。
- 能指出本节涉及的订单字段、排序字段、事件或退款字段来自哪个 Move 模块。
- 能用一笔具体订单解释价格优先、时间优先、成交、挂单、取消或查询结果如何产生。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：`validate_inputs`、`match_maker`、`assert_execution` 和订单生命周期事件。
- [packages/deepbook/sources/book/fill.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/fill.move)：`Fill` 的 maker 方向、成交数量、completed/expired 和结算字段。

## 正文

订单类型在 `constants.move` 中包括：

- `no_restriction()`：可吃单，剩余可挂单。
- `immediate_or_cancel()`：可吃单，剩余立即取消，不挂单。
- `fill_or_kill()`：必须完全成交，否则 abort。
- `post_only()`：不得与现有订单成交，否则 abort。

`order_info::assert_execution` 负责检查这些限制。post-only 如果 `executed_quantity > 0` 会 abort；FOK 如果未完全成交会 abort；IOC 会把剩余数量标成 canceled 并返回，不插入订单簿。

自成交策略在 `order_info::match_maker`：

- `cancel_taker()`：如果 maker 和 taker 使用同一个 BalanceManager，直接 abort。
- `cancel_maker()`：如果同账户相遇，生成 expired/canceled maker fill，并移除 maker。
- 默认允许自成交。

机器人和做市系统必须明确设置 self matching option，否则同一个 BalanceManager 下的多个策略可能互相成交。

补充阅读：阅读 post-only、IOC、FOK、自成交策略 时，先从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的入口确认交易类型，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消路径。taker 的即时执行状态保存在 `OrderInfo`，maker 的可挂单状态保存在 `Order`，这两个对象不要混在一起看。

函数调用链可以按“入口参数校验 -> `OrderInfo::validate_inputs` -> `book::match_against_book` -> `OrderInfo::assert_execution`”阅读。资金路径不是在 `Book` 内直接转 coin，而是由成交结果写入账户应收应付，后续再交给 Vault 和 BalanceManager 结算。

## 开发要点

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 检查问题

- post-only、IOC、FOK、自成交策略 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
