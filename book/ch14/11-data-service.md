# ch14-11 数据服务

[返回本章](README.md)

## 先定义产品场景

产品小节先想用户路径和失败路径。DeepBook 应用的难点往往不在单个 move call，而在状态同步、错误解释和风险降级。

## 源码入口

- [crates/server/src/server.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/server.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/reader.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/reader.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。
- [crates/server/src/metrics/middleware.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/metrics/middleware.rs)：REST API、数据库查询、健康检查或指标实现。

## 把系统拼起来

数据服务从 DeepBook Server 或本地 PostgreSQL 读取数据，向前端输出更稳定的 DTO。它负责缓存、分页、限流和聚合，不负责资金状态修改。


数据服务负责把 Indexer 表转换成应用级 API。每个端点都要定义过滤条件、分页、缓存时间、错误码和指标；机器人使用的数据端点还要暴露数据新鲜度。

## 产品落地判断

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 动手检查

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
