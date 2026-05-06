# s04 flashloans 表记录

目标：展示 indexer 如何把 `FlashLoanBorrowed` 写入 PostgreSQL 的 `flashloans` 表。

源码：

- `crates/indexer/src/handlers/flash_loan_handler.rs`

映射字段：

```text
event_digest
digest
sender
checkpoint
checkpoint_timestamp_ms
package
pool_id
borrow_quantity
borrow
type_name
```

查询示例：

```sql
select
  checkpoint_timestamp_ms,
  sender,
  pool_id,
  type_name,
  borrow_quantity
from flashloans
where pool_id = $1
order by checkpoint desc
limit 50;
```

环境变量：

```text
DATABASE_URL=postgres://user:pass@localhost:5432/deepbook
```

注意事项：

- `borrow` 当前由 handler 固定写为 `true`。
- 没有独立还款行；成功交易表示 hot potato 已被归还函数消费。
