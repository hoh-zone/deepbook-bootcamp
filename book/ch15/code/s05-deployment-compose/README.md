# s05 deployment compose

部署组件：

- PostgreSQL
- deepbook-indexer
- deepbook-server
- Prometheus
- Grafana
- frontend

生产环境需要持久化数据库和指标数据，不要使用临时 volume。

## 建议命令

```bash
docker compose config
docker compose up -d postgres deepbook-indexer deepbook-server
docker compose ps
docker compose logs -f deepbook-indexer
```

## 必备配置

```bash
DATABASE_URL=postgres://...
SUI_RPC_URL=https://...
DEEPBOOK_PACKAGES=deepbook,margin
FIRST_CHECKPOINT=...
RUST_LOG=info
```

## 验收

- PostgreSQL 有持久化 volume 和备份策略。
- Indexer checkpoint lag 可观测。
- Server 有 `/status` 或等价健康检查。
- Prometheus/Grafana 不和生产数据库共享高权限账号。
