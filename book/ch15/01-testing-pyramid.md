# ch15-01 测试金字塔

[返回本章](README.md)

## 先定交付标准

这一节按上线视角读“测试金字塔”。如果某个判断不能被测试、监控或运行手册证明，它就还不是出版级和生产级内容。

## 源码入口

- [packages/deepbook/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/tests)：测试 fixture、断言或可复现验证材料。
- [packages/deepbook_margin/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/tests)：测试 fixture、断言或可复现验证材料。
- [packages/predict/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/tests)：测试 fixture、断言或可复现验证材料。
- [crates/indexer/tests/snapshot_tests.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/snapshot_tests.rs)：测试 fixture、断言或可复现验证材料。

## 从测试到上线

DeepBook 开发测试分四层：Move 单测验证合约不变量，SDK 单测验证交易构造，Indexer 快照测试验证事件落库，端到端测试验证从 UI 到链上再回到数据层的闭环。


测试金字塔从 Move 单元测试开始，向上覆盖 SDK/PTB、Indexer snapshot、Server API 和前端端到端。越靠近底层越应该快且确定，越靠近应用越应覆盖真实用户路径和故障提示。

## 交付判断

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 动手检查

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
