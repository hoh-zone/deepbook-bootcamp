# ch15-18 附录

[返回本章](README.md)

## 本节目标

- 明确“附录”在 DeepBook 测试、安全、部署或交付体系中的验收目标。
- 能定位对应的 Move 测试、Indexer snapshot、Server 指标、Docker 或运行手册路径。
- 能把本节要求转化为可复现的测试证据、审计材料或生产检查项。

## 源码关联

- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构、索引、Diesel schema 或迁移约束。
- [crates/server/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/README.md)：REST API、数据库查询、健康检查或指标实现。
- [packages/deepbook/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/tests)：测试 fixture、断言或可复现验证材料。
- [packages/deepbook_margin/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/tests)：测试 fixture、断言或可复现验证材料。

## 正文

附录要服务查阅：函数速查、事件速查、表结构、错误码、SDK 片段、术语中英文对照、出版前事实核对。


附录适合收录术语表、路径索引、错误码、表结构摘要和命令备忘。它应服务查阅，而不是承载新概念；正文中出现的关键名词要能在附录中快速反查。

## 开发要点

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 检查问题

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
