# ch15-06 前端端到端测试

[返回本章](README.md)

## 先定交付标准

这一节先看验收证据：测试、日志、监控、部署步骤、审计材料或发布前核对项必须能复现。

## 源码入口

- [crates/server/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/README.md)：REST API、数据库查询、健康检查或指标实现。
- [packages/deepbook/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/tests)：测试 fixture、断言或可复现验证材料。
- [packages/deepbook_margin/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/tests)：测试 fixture、断言或可复现验证材料。
- [packages/predict/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/tests)：测试 fixture、断言或可复现验证材料。

## 从测试到上线

端到端测试覆盖钱包连接、下单、签名、提交、确认、订单刷新、撤单和错误提示。交易状态机是重点，不是按钮是否能点击。


端到端测试应从用户路径出发：连接钱包、选择池子、模拟、签名、提交、确认、等待 Indexer 可见和错误恢复。测试环境要能注入 API 延迟、余额不足和 checkpoint lag。

## 交付判断

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 动手检查

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
