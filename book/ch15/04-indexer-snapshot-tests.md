# ch15-04 Indexer snapshot tests

[返回本章](README.md)

## 本节目标

- 明确“Indexer 快照测试”在 DeepBook 测试、安全、部署或交付体系中的验收目标。
- 能定位对应的 Move 测试、Indexer snapshot、Server 指标、Docker 或运行手册路径。
- 能把本节要求转化为可复现的测试证据、审计材料或生产检查项。

## 源码关联

- [crates/indexer/tests/snapshot_tests.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/snapshot_tests.rs)：测试 fixture、断言或可复现验证材料。
- [crates/indexer/tests/checkpoints/order_fill/100000337.chk](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/checkpoints/order_fill/100000337.chk)：测试 fixture、断言或可复现验证材料。
- [crates/indexer/tests/checkpoints/loan_borrowed/273619256.chk](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/checkpoints/loan_borrowed/273619256.chk)：测试 fixture、断言或可复现验证材料。
- [crates/indexer/tests/snapshots/snapshot_tests__flash_loans__flashloans.snap](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/snapshots/snapshot_tests__flash_loans__flashloans.snap)：测试 fixture、断言或可复现验证材料。

## 正文

`crates/indexer/tests/snapshot_tests.rs` 使用真实 checkpoint fixture 验证 handler 输出。它能防止事件结构、字段映射和表结构变化后静默破坏 API。


Snapshot tests 用真实 checkpoint 固定 handler 的输出合同。新增事件或改字段时要同时更新 checkpoint fixture、预期 snapshot、schema model，并检查 `flashloans` 与 Margin `loan_borrowed`、`loan_repaid` 没有混表。

## 开发要点

- 每个测试或部署结论都要能回到具体命令、fixture、日志、指标或审计材料。
- 同时覆盖成功路径、失败路径、边界值、权限错误和数据延迟。
- 升级、迁移和部署步骤要写明回滚点、兼容窗口、健康检查和负责人。
- 涉及资金安全时，把链上约束、SDK 构造、Indexer 读模型和前端提示分层验证。

## 检查问题

- 本节的验收证据是 Move 测试、SDK 测试、snapshot、E2E、指标还是部署日志？
- 哪些失败路径、权限边界、迁移风险或运维故障还没有被覆盖？
- 如果生产事故发生，能否用 digest、checkpoint、日志和监控在 10 分钟内定位责任层？
