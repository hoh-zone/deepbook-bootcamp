# ch04-07 post-only、IOC、FOK、自成交策略

[返回本章](README.md)

## 先看订单现场

读这一节时，不要从函数名开始。先问一笔订单走到“post-only、IOC、FOK、自成交策略”这个环节时，排序、成交、挂单、取消或退款中哪一个状态会发生变化。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：`validate_inputs`、`match_maker`、`assert_execution` 和订单生命周期事件。
- [packages/deepbook/sources/book/fill.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/fill.move)：`Fill` 的 maker 方向、成交数量、completed/expired 和结算字段。

## 关键定义

订单执行策略和自成交策略都是 `u8` 常量函数。SDK、前端和策略程序最好通过官方 SDK 或同名常量映射生成参数，不要在业务代码里散落裸数字。

```move
public fun no_restriction(): u8 {
    NO_RESTRICTION
}

public fun immediate_or_cancel(): u8 {
    IMMEDIATE_OR_CANCEL
}

public fun fill_or_kill(): u8 {
    FILL_OR_KILL
}

public fun post_only(): u8 {
    POST_ONLY
}

public fun self_matching_allowed(): u8 {
    SELF_MATCHING_ALLOWED
}

public fun cancel_taker(): u8 {
    CANCEL_TAKER
}

public fun cancel_maker(): u8 {
    CANCEL_MAKER
}
```

Move 里用常量函数而不是枚举，是为了让入口参数保持简单。但这也意味着应用层需要主动做参数约束：`post_only` 和市价单不兼容，`fill_or_kill` 需要先用订单簿估算完整成交概率，自成交策略需要和同一个 `BalanceManager` 下的多策略机器人配合。

## 把订单走一遍

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

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

函数调用链可以按“入口参数校验 -> `OrderInfo::validate_inputs` -> `book::match_against_book` -> `OrderInfo::assert_execution`”阅读。资金路径不是在 `Book` 内直接转 coin，而是由成交结果写入账户应收应付，后续再交给 Vault 和 BalanceManager 结算。

## 交易实现提醒

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 动手检查

- post-only、IOC、FOK、自成交策略 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
