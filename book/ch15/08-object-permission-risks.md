# ch15-08 对象权限风险

[返回本章](README.md)

## 本节目标

- 明确“对象权限风险”在 DeepBook 测试、安全、部署或交付体系中的验收目标。
- 能定位对应的 Move 测试、Indexer snapshot、Server 指标、Docker 或运行手册路径。
- 能把本节要求转化为可复现的测试证据、审计材料或生产检查项。

## 源码关联

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [packages/deepbook_margin/sources/margin_registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_registry.move)：Margin 合约对象、借贷、抵押、清算或风控逻辑。
- [packages/predict/sources/predict_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/predict_manager.move)：Predict 市场、oracle、定价、结算或测试逻辑。

## 正文

Sui Move 中 capability 是权限边界。管理员 cap、maintainer cap、supplier cap、pause cap 和 trade proof 都必须明确拥有者、传递路径和失效方式。


对象权限风险来自 shared object、capability、BalanceManager 和 registry 的组合。测试要证明非授权账户不能修改池子配置、提取他人余额、篡改 oracle 或绕过 Margin 风控。

## 开发要点

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 检查问题

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
