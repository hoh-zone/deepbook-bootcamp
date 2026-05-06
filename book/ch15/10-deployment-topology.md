# ch15-10 部署拓扑

[返回本章](README.md)

## 先定交付标准

这一节先看验收证据：测试、日志、监控、部署步骤、审计材料或发布前核对项必须能复现。

## 源码入口

- [docker/deepbook-indexer/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-indexer/Dockerfile)：容器构建、入口脚本或部署边界。
- [docker/deepbook-server/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-server/Dockerfile)：REST API、数据库查询、健康检查或指标实现。
- [docker/deepbook-server/entry.sh](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-server/entry.sh)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/main.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/main.rs)：REST API、数据库查询、健康检查或指标实现。

## 从测试到上线

推荐生产拓扑：前端、交易构造服务、DeepBook Server、DeepBook Indexer、PostgreSQL、Prometheus、Grafana、日志系统。Indexer 与 Server 分开部署，便于独立扩缩容。


部署拓扑至少包含 Indexer、Server、PostgreSQL、RPC、metrics 和告警。Indexer 与 Server 不应共享同一权限配置；数据库迁移、备份和只读连接池也要拆开管理。

## 交付判断

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 动手检查

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
