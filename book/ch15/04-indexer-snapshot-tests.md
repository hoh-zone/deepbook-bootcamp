# ch15-04 Indexer snapshot tests

[返回本章](README.md)

## 先定交付标准

Indexer snapshot test 是数据系统的合约测试。Move 事件、Rust handler、Diesel model、PostgreSQL schema 和 Server API 中任何一层变动，都可能让前端看到不同字段或错误数值。Snapshot 的价值就是把这些变化显性化。

## 源码入口

- [crates/indexer/tests/snapshot_tests.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/snapshot_tests.rs)：测试 fixture、断言或可复现验证材料。
- [crates/indexer/tests/checkpoints/order_fill/100000337.chk](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/checkpoints/order_fill/100000337.chk)：测试 fixture、断言或可复现验证材料。
- [crates/indexer/tests/checkpoints/loan_borrowed/273619256.chk](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/checkpoints/loan_borrowed/273619256.chk)：测试 fixture、断言或可复现验证材料。
- [crates/indexer/tests/snapshots/snapshot_tests__flash_loans__flashloans.snap](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/snapshots/snapshot_tests__flash_loans__flashloans.snap)：测试 fixture、断言或可复现验证材料。

## 从测试到上线

`crates/indexer/tests/snapshot_tests.rs` 使用真实 checkpoint fixture 验证 handler 输出。它能防止事件结构、字段映射和表结构变化后静默破坏 API。

Snapshot tests 固定的是读模型合同。新增事件或改字段时，要同时更新 checkpoint fixture、预期 snapshot、schema model，并检查 `flashloans` 与 Margin `loan_borrowed`、`loan_repaid` 没有混表。对交易终端来说，这不是后端细节，而是用户历史、K 线、资金流水和风险面板的事实来源。

一套合格的 snapshot 测试还要记录 fixture 来源、checkpoint 编号、事件类型、预期表和关键字段含义。否则测试虽然能跑，后来的人也不知道它保护的是哪条业务路径。

## 交付判断

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。
- 每个 snapshot 都要能解释它保护的业务查询，例如成交流、闪电贷历史、Margin 借款或清算列表。

## 动手检查

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
