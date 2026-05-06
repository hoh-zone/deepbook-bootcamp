# s05 api client

DeepBook Server 常用端点：

```bash
curl http://localhost:9008/status
curl http://localhost:9008/get_pools
curl http://localhost:9008/ticker
curl http://localhost:9008/trades/SUI_USDC
curl http://localhost:9008/orderbook/SUI_USDC
```

前端不要把所有接口设为同一刷新频率。ticker 和 orderbook 可以快，pool 配置和 fees 可以慢。

