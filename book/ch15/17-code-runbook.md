# ch15-17 代码运行手册

[返回本章](README.md)

## 先定交付标准

这一节按上线视角读“代码运行手册”。如果某个判断不能被测试、监控或运行手册证明，它就还不是出版级和生产级内容。

## 源码入口

- `book/ch13/code/s02-run-indexer/README.md`：本书配套示例、运行模板或验收材料。
- `book/ch14/code/s03-arbitrage-bot/README.md`：本书配套示例、运行模板或验收材料。
- `book/ch15/code/s04-e2e-test-template/README.md`：本书配套示例、运行模板或验收材料。
- [docker/deepbook-server/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-server/Dockerfile)：REST API、数据库查询、健康检查或指标实现。

## 从测试到上线

运行手册按章节组织，不要求读者一次启动所有服务。Spot、Margin、Predict、Indexer 应分别给出最小运行路径。


代码运行手册要按“准备、启动、验证、停止、清理”组织。每个示例都应列出必要环境变量、网络、端口、预期 API 响应和失败排查入口。

## 交付判断

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 动手检查

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
