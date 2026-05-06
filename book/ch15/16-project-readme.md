# ch15-16 全书项目 README

[返回本章](README.md)

## 本节目标

- 明确“项目 README”在 DeepBook 测试、安全、部署或交付体系中的验收目标。
- 能定位对应的 Move 测试、Indexer snapshot、Server 指标、Docker 或运行手册路径。
- 能把本节要求转化为可复现的测试证据、审计材料或生产检查项。

## 源码关联

- `book/ch13/code/s01-local-postgres/README.md`：本书配套示例、运行模板或验收材料。
- `book/ch14/code/s01-trading-terminal/README.md`：本书配套示例、运行模板或验收材料。
- `book/ch15/code/s05-deployment-compose/README.md`：本书配套示例、运行模板或验收材料。
- [docker](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker)：容器构建、入口脚本或部署边界。

## 正文

最终 README 应说明阅读路径、环境要求、代码运行顺序、源码仓库版本、测试命令和常见排障。


全书项目 README 要把环境准备、章节顺序、运行命令、依赖版本和故障排查集中起来。读者不应靠猜测找到 PostgreSQL、Indexer、Server、前端和测试模板的启动顺序。

## 开发要点

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 检查问题

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
