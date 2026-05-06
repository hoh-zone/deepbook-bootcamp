# ch13-05 Handler 与 `map_event` 模式

[返回本章](README.md)

## 先看数据问题

这一节从读模型而不是数据库表开始。围绕“Handler 与 map_event 模式”，先问交易终端、机器人或风控系统要查什么，再看 handler 和 schema 如何支撑。

## 源码入口

- [crates/indexer/src/handlers/mod.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/mod.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/indexer/src/handlers/order_fill_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/order_fill_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/indexer/src/handlers/flash_loan_handler.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers/flash_loan_handler.rs)：事件解析、字段映射或业务流水落库入口。
- [crates/schema/src/models.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/models.rs)：表结构、索引、Diesel schema 或迁移约束。

## 关键定义

以成交事件 handler 为例：

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
        pool_id: event.pool_id.to_string(),
        maker_order_id: event.maker_order_id.to_string(),
        taker_order_id: event.taker_order_id.to_string(),
        price: event.price as i64,
        base_quantity: event.base_quantity as i64,
        quote_quantity: event.quote_quantity as i64,
        onchain_timestamp: event.timestamp as i64,
    }
}
```

这段定义把 Indexer 的职责说清楚了：`event_type` 指链上事件，`db_model` 指 Rust/Diesel model，`table` 指 PostgreSQL 表，`map_event` 只做字段映射。不要在 handler 里做复杂业务聚合；K 线、用户历史、订单状态聚合应该在 Server 查询层或物化任务中完成。

## 落到读模型里

`define_handler!` 宏把重复逻辑固定下来：遍历 checkpoint 中的交易，筛选 DeepBook 相关交易，匹配事件类型，用 BCS 反序列化，再映射成数据库 model，最后批量 insert。

以 `OrderFillHandler` 为例，`OrderFilled` 事件会被映射成 `OrderFill`：

- `pool_id`：成交所在池子。
- `maker_order_id`、`taker_order_id`：双方订单。
- `price`、`base_quantity`、`quote_quantity`：成交价格和数量。
- `maker_fee`、`taker_fee`：双方费用。
- `maker_balance_manager_id`、`taker_balance_manager_id`：双方交易账户。

这就是行情、成交历史和用户交易记录的基础表。


`define_handler!` 把“事件类型、目标表、字段映射”放在一个局部声明中。`map_event` 不应该做复杂查询，它只把链上事件和 `EventMeta` 组合成 Diesel insert model；需要聚合的订单簿、K 线和用户历史应放到 Server 查询或离线物化任务。

## 数据系统判断

- 先确认本节涉及的事件名、handler、目标表和 API 端点，避免把链上对象状态与读模型混用。
- 查询设计必须带池子、账户、checkpoint 或时间窗口，并说明分页和索引依据。
- 对实时应用要同时检查 `/status`、checkpoint lag 和数据时间戳，再决定是否交易、降级或告警。
- 涉及 Flash Loan 或 Margin 时，明确 `flashloans` 与 `loan_borrowed`、`loan_repaid` 的语义差异。

## 动手检查

- 本节对应的 handler、表名和 Server 查询入口分别是什么？
- 这个数据在链上、Indexer、PostgreSQL、Server 缓存和前端之间可能出现哪些延迟或不一致？
- 如果用于机器人或风控，应该设置什么 checkpoint lag、分页窗口和降级策略？
