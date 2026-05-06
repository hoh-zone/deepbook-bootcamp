# s05 deployment compose

部署组件：

- PostgreSQL
- deepbook-indexer
- deepbook-server
- Prometheus
- Grafana
- frontend

生产环境需要持久化数据库和指标数据，不要使用临时 volume。

