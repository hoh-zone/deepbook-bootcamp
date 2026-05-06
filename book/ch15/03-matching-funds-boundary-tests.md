# ch15-03 撮合和资金边界测试

[返回本章](README.md)

## 先定交付标准

这一节按上线视角读“撮合和资金边界测试”。如果某个判断不能被测试、监控或运行手册证明，它就还不是出版级和生产级内容。

## 源码入口

- [packages/deepbook/tests/book/order_tests.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/tests/book/order_tests.move)：测试 fixture、断言或可复现验证材料。
- [packages/deepbook/tests/pool_tests.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/tests/pool_tests.move)：测试 fixture、断言或可复现验证材料。
- [packages/deepbook_margin/tests/margin_pool/margin_state_tests.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/tests/margin_pool/margin_state_tests.move)：测试 fixture、断言或可复现验证材料。
- [packages/deepbook_margin/tests/margin_manager_borrow_share_tests.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/tests/margin_manager_borrow_share_tests.move)：测试 fixture、断言或可复现验证材料。

## 从测试到上线

撮合测试关注价格优先、时间优先、部分成交、完全成交、撤单和自成交边界。资金测试关注余额锁定、结算、费用、返佣和 Vault 余额守恒。


撮合和资金边界测试要验证部分成交、取消、余额占用、费用和借贷份额的守恒。失败路径同样重要，例如余额不足、价格越界、权限对象错误和清算阈值附近的舍入。

## 交付判断

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 动手检查

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
