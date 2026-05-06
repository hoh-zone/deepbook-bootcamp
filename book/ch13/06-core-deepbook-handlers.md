# ch13-06 Core DeepBook handlers

[返回本章](README.md)

## 先看数据问题

链上事件本身不适合直接支撑交易终端。用户要看订单状态、成交流、余额历史和行情，机器人还要按 checkpoint 延迟决定是否暂停交易。这些都需要读模型。

所以这一节不从 PostgreSQL 表名开始，而是从 handler 的职责开始：每个 handler 把一种链上事件翻译成一张可以查询、分页、对账和监控的表。

## 源码入口

- [crates/indexer/src/handlers/order_fill_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_fill_handler.rs)：成交流和 K 线的基础。
- [crates/indexer/src/handlers/order_update_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_update_handler.rs)：订单生命周期的基础。
- [crates/indexer/src/handlers/balances_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/balances_handler.rs)：BalanceManager 资金历史的基础。
- [crates/indexer/src/handlers/pool_price_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/pool_price_handler.rs)：参考价格和行情缓存的基础。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。

## 关键定义

Indexer 的核心 handler 不是手写一堆重复 trait impl，而是通过 `define_handler!` 宏声明事件类型、目标表和字段映射。下面是成交事件的典型形态。

```rust
define_handler! {
    name: OrderFillHandler,
    processor_name: "order_fill",
    event_type: OrderFilled,
    db_model: OrderFill,
    table: order_fills,
    map_event: |event, meta| OrderFill {
        event_digest: meta.event_digest(),
        digest: meta.digest(),
        sender: meta.sender(),
        checkpoint: meta.checkpoint(),
        checkpoint_timestamp_ms: meta.checkpoint_timestamp_ms(),
        package: meta.package(),
        pool_id: event.pool_id.to_string(),
        maker_order_id: event.maker_order_id.to_string(),
        taker_order_id: event.taker_order_id.to_string(),
        price: event.price as i64,
        taker_is_bid: event.taker_is_bid,
        base_quantity: event.base_quantity as i64,
        quote_quantity: event.quote_quantity as i64,
        onchain_timestamp: event.timestamp as i64,
    }
}
```

这段映射体现了读模型的取舍：`event_digest`、`digest`、`checkpoint` 是可追溯性字段；`pool_id`、order id 和数量字段是业务查询字段。写查询 API 时不要只按业务字段建索引，还要保留 checkpoint 维度，便于补扫、对账和延迟监控。

## 落到读模型里

核心 DeepBook pipeline 可以按产品问题来记：

- `OrderFillHandler`：写入 `order_fills`。
- `OrderUpdateHandler`：写入 `order_updates`，用于订单状态。
- `BalancesHandler`：写入 `balances`，用于 BalanceManager 资金历史。
- `PoolCreatedHandler`：写入池子创建事件。
- `PoolPriceHandler`：写入参考价格。
- `TradeParamsUpdateHandler`：写入交易参数变化。
- `StakesHandler`、`RebatesHandler`、`RebatesV2Handler`：写入 DEEP stake 和返佣数据。
- `BookParamsUpdatedHandler`：写入 tick、lot 等 book 参数变化。

交易终端通常至少需要 `order_fills`、`order_updates`、`balances`、`pools` 和 `pool_prices`。`order_fills` 生成成交流和 K 线，`order_updates` 跟踪订单生命周期，`balances` 解释 BalanceManager 维度的资金变化，`pool_prices` 支撑行情展示和风控提示。

## 数据系统判断

- 查询接口不要只按业务字段设计，还要带 checkpoint 或时间窗口，方便补扫和对账。
- 实时交易应用必须同时看 `/status`、checkpoint lag 和数据时间戳，再决定是否继续自动交易。
- Flash Loan、Margin loan 和普通成交不要混表解释；它们的生命周期完全不同。

## 动手检查

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
