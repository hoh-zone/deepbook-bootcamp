# ch15-05 SDK 测试

[返回本章](README.md)

## 先定交付标准

这一节按上线视角读“SDK 测试”。如果某个判断不能被测试、监控或运行手册证明，它就还不是出版级和生产级内容。

## 源码入口

- [scripts/transactions](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions)：TypeScript 交易构造或运行脚本样例。
- [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)：Margin 合约对象、借贷、抵押、清算或风控逻辑。
- [packages/predict/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/tests)：测试 fixture、断言或可复现验证材料。

## 从测试到上线

SDK 测试分为交易构造测试和 dry run 测试。构造测试检查 PTB 目标函数、对象参数和类型参数；dry run 测试检查链上执行是否会 abort。


SDK 测试要覆盖 PTB 构造、type arguments、对象输入和 dry run 错误映射。它不只验证函数能调用，还要验证调用者拿到的 digest、返回对象和错误提示能驱动前端状态机。

## 交付判断

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 动手检查

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
