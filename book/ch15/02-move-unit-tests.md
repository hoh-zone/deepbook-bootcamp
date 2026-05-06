# ch15-02 Move 单元测试

[返回本章](README.md)

## 本节目标

- 明确“Move 测试”在 DeepBook 测试、安全、部署或交付体系中的验收目标。
- 能定位对应的 Move 测试、Indexer snapshot、Server 指标、Docker 或运行手册路径。
- 能把本节要求转化为可复现的测试证据、审计材料或生产检查项。

## 源码关联

- [packages/deepbook/tests/pool_tests.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/tests/pool_tests.move)：测试 fixture、断言或可复现验证材料。
- [packages/deepbook/tests/vault/vault_tests.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/tests/vault/vault_tests.move)：测试 fixture、断言或可复现验证材料。
- [packages/deepbook_margin/tests/margin_manager_tests.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/tests/margin_manager_tests.move)：测试 fixture、断言或可复现验证材料。
- [packages/predict/tests/config/pricing_config_tests.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/tests/config/pricing_config_tests.move)：测试 fixture、断言或可复现验证材料。

## 正文

Move 测试应覆盖正常路径、边界条件和 expected failure。金融协议测试不能只验证成功交易，还要验证错误权限、错误资产、错误数量、版本错误和资金不足。


Move 单元测试负责证明对象权限、资金守恒、错误码和边界值。DeepBook、Margin、Predict 的测试应分别覆盖撮合、闪电贷归还、借贷清算和 oracle/定价规则。

## 开发要点

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 检查问题

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
