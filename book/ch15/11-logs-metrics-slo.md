# ch15-11 日志、指标和 SLO

[返回本章](README.md)

## 先定交付标准

这一节按上线视角读“日志、指标和 SLO”。如果某个判断不能被测试、监控或运行手册证明，它就还不是出版级和生产级内容。

## 源码入口

- [crates/server/src/metrics/mod.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/metrics/mod.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/metrics/middleware.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/metrics/middleware.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/src/margin_metrics/metrics.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/margin_metrics/metrics.rs)：REST API、数据库查询、健康检查或指标实现。
- [crates/server/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/README.md)：REST API、数据库查询、健康检查或指标实现。

## 从测试到上线

日志必须包含 digest、checkpoint、pool、balance manager、endpoint、错误码。SLO 至少覆盖 API 延迟、Indexer lag、交易提交成功率和数据刷新延迟。


SLO 应以用户可感知结果定义，例如 API P95、错误率、checkpoint lag、Margin 轮询成功率和交易确认耗时。日志必须带 digest、pool_id、balance_manager_id 或 checkpoint，方便从 UI 问题追到链上和数据库。

## 交付判断

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 动手检查

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
