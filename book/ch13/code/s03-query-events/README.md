# s03 query events

常用查询：

```sql
select pool_id, price, base_quantity, quote_quantity, checkpoint_timestamp_ms
from order_fills
order by checkpoint_timestamp_ms desc
limit 20;

select pool_id, type_name, borrow_quantity, checkpoint_timestamp_ms
from flashloans
order by checkpoint_timestamp_ms desc
limit 20;

select margin_manager_id, margin_pool_id, loan_amount, checkpoint_timestamp_ms
from loan_borrowed
order by checkpoint_timestamp_ms desc
limit 20;
```

`flashloans` 是 DeepBookV3 Vault 的同交易借还能力；`loan_borrowed` 是 Margin 跨交易债务事件。

