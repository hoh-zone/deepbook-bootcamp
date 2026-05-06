# s04 Service Health Checks

本目录保存 DeepBook Sandbox 常用服务检查命令。

## 端口

| 服务 | 地址 |
| --- | --- |
| Dashboard | `http://localhost:5173` |
| Sui RPC | `http://localhost:9000` |
| Sui Faucet | `http://localhost:9123` |
| DeepBook Faucet | `http://localhost:9009` |
| DeepBook Server | `http://localhost:9008` |
| Oracle | `http://localhost:9010` |
| Market Maker | `http://localhost:3001/health` |
| Market Maker Metrics | `http://localhost:9091/metrics` |

## 检查命令

```bash
docker compose ps
curl http://localhost:9010/
curl http://localhost:3001/health
curl http://localhost:9009/
curl http://localhost:9009/manifest
curl http://localhost:9091/metrics
```

## 日志

```bash
docker compose logs -f
docker compose logs -f oracle-service
docker compose logs -f market-maker
docker compose logs -f deepbook-indexer
docker compose logs -f deepbook-server
```

如果 dashboard 失败，先用这些命令定位是前端代理、RPC、faucet、oracle、market maker、indexer 还是 server 的问题。
