# ch04-06 市价单和吃单资金检查

[返回本章](README.md)

## 本节目标

- 明确 市价单和吃单资金检查 在 `Pool -> Book -> OrderInfo/Order` 撮合链中的职责。
- 能指出本节涉及的订单字段、排序字段、事件或退款字段来自哪个 Move 模块。
- 能用一笔具体订单解释价格优先、时间优先、成交、挂单、取消或查询结果如何产生。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：`validate_inputs`、`match_maker`、`assert_execution` 和订单生命周期事件。

## 正文

`pool::place_market_order` 不是独立撮合逻辑，而是调用 `place_order_int`：

- bid 市价单使用 `constants::max_price()`。
- ask 市价单使用 `constants::min_price()`。
- order type 固定为 `immediate_or_cancel()`。
- `market_order = true`，并把 expire timestamp 设为当前时间。

`order_info::validate_inputs` 对 market order 跳过普通限价价格校验，但禁止 post-only。资金检查最终体现在 `order_info.calculate_partial_fill_balances` 和 `vault.settle_balance_manager`：买方 owed quote 和 fee，卖方 owed base 和 fee。如果 BalanceManager 余额不足，`withdraw_with_proof` 路径会 abort。

生产实践：市价单应先用 `book::get_quantity_out` 或 SDK quote 能力估算成交量和 fee，并设置最小输出保护。仅靠 max/min price 会暴露滑点风险。

补充阅读：阅读 市价单和吃单资金检查 时，先从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的入口确认交易类型，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消路径。taker 的即时执行状态保存在 `OrderInfo`，maker 的可挂单状态保存在 `Order`，这两个对象不要混在一起看。

函数调用链可以按“入口参数校验 -> `OrderInfo::validate_inputs` -> `book::match_against_book` -> `OrderInfo::assert_execution`”阅读。资金路径不是在 `Book` 内直接转 coin，而是由成交结果写入账户应收应付，后续再交给 Vault 和 BalanceManager 结算。

## 开发要点

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 检查问题

- 市价单和吃单资金检查 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
