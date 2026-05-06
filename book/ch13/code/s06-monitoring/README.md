# s06 monitoring

最小健康检查：

```bash
curl "http://localhost:9008/status?max_checkpoint_lag=100&max_time_lag_seconds=60"
```

告警建议：

- `status != OK`
- `max_checkpoint_lag > 100`
- `max_time_lag_seconds > 60`
- Server 5xx 升高
- PostgreSQL 连接池耗尽

