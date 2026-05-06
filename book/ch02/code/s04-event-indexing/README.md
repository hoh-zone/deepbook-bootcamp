# s04-event-indexing 事件示例

目标：实现一个会发出事件的 Move 函数，并用 RPC 或 Indexer 思路理解事件如何进入链下数据系统。

## 事件结构

```move
public struct CounterIncremented has copy, drop {
    counter_id: ID,
    old_value: u64,
    new_value: u64,
}
```

在 `increment` 中调用 `event::emit(CounterIncremented { ... })`。DeepBook 中的 `OrderFilled`、`OrderUpdate`、`FlashLoanBorrowed`、`LoanBorrowed` 都使用同样的事件驱动思想，只是字段更复杂。

## 查询路径

1. 提交交易。
2. 记录 transaction digest。
3. 通过 Sui RPC 查询事件。
4. 对照 `crates/indexer/src/handlers/mod.rs` 理解生产 Indexer 如何把事件写入 PostgreSQL。

