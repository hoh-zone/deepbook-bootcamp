# s04 kline builder

K 线来自 `order_fills`。核心字段：

- `price`
- `base_quantity`
- `quote_quantity`
- `checkpoint_timestamp_ms`
- `pool_id`

实现步骤：

1. 按 `pool_id` 过滤。
2. 把 `checkpoint_timestamp_ms` 截断到固定时间桶。
3. 每个桶计算 open、high、low、close、base volume、quote volume。
4. 高频请求使用预聚合表或物化视图。

查询骨架：

```sql
select
  pool_id,
  date_trunc('minute', to_timestamp(checkpoint_timestamp_ms / 1000.0)) as bucket,
  min(price) as low,
  max(price) as high,
  sum(base_quantity) as base_volume,
  sum(quote_quantity) as quote_volume
from order_fills
where pool_id = $1
group by pool_id, bucket
order by bucket desc
limit 100;
```

本地验证：

```bash
psql "$DATABASE_URL" -f kline.sql
```
