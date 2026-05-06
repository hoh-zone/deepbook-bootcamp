# ch15-07 金融协议安全清单

[返回本章](README.md)

## 本节目标

- 明确“金融安全清单”在 DeepBook 测试、安全、部署或交付体系中的验收目标。
- 能定位对应的 Move 测试、Indexer snapshot、Server 指标、Docker 或运行手册路径。
- 能把本节要求转化为可复现的测试证据、审计材料或生产检查项。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)：Margin 合约对象、借贷、抵押、清算或风控逻辑。
- [packages/deepbook_margin/sources/rate_limiter.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/rate_limiter.move)：Margin 合约对象、借贷、抵押、清算或风控逻辑。
- [packages/predict/sources/config/risk_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/config/risk_config.move)：Predict 市场、oracle、定价、结算或测试逻辑。

## 正文

必须检查：权限、capability 泄漏、资产类型混淆、精度错误、溢出、预言机过期、价格偏离、重复执行、状态不同步、Indexer 延迟和交易失败后的 UI 状态。


金融协议安全清单应覆盖资金守恒、价格源、权限、限速、清算、费用和紧急暂停。每一项都要能指向测试或监控证据，不能只停留在“已审查”的文本声明。

## 开发要点

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 检查问题

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
