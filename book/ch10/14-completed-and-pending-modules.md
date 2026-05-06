# ch10-14 已完成和待完成模块

[返回本章](README.md)

## 本节目标

- 汇总 Predict 当前已完成、源码可见和仍处迁移计划的模块。
- 为文档、SDK 和应用标注明确的版本/网络状态。
- 建立继续开发时的验收清单，避免把计划项写成事实。

## 源码关联

- [PREDICT_MIGRATION.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/PREDICT_MIGRATION.md)：PR 状态和 Phase 2 计划。
- `packages/predict/sources/*`：本书采用的 GitHub 源码快照中的 Move 模块。
- `packages/predict/simulations/*`：本地仿真 runtime、scenario 和可视化脚本。

## 正文

按 `PREDICT_MIGRATION.md`，已完成：Oracle、constants/math、Vault、pricing config、risk config。迁移计划中待完成：Market Key、PredictManager、Predict Core、Registry、单元测试、集成测试、Predict schema、Predict Indexer、Predict Server、部署脚本、oracle services、Docker 和 CI。

按本书采用的 GitHub 源码快照，可阅读和本地开发环境试验：`range_key.move`、`predict_manager.move`、`predict.move`、`registry.move`、`oracle_config.move`、`fee_reserve.move`、仿真 runtime。书稿后续章节引用这些文件时，必须加上“本书采用的 GitHub 源码快照已有，迁移计划仍未全部标为完成”的限定。

可以把模块分成三层：迁移文档已合入的链上基础、工作树中可读可仿真的 Move/Simulation、尚未完成的 Indexer/Server/部署脚本/Oracle services。只有第一层能被写成已合入；第二层适合精读和本地实验；第三层只能写设计边界。

SDK 或服务配置建议包含 `predictVersionStatus`：例如 `migration-pr1-pr2-merged-local-snapshot`、`localnet-simulation-only`、`mainnet-unverified`。缺少该字段时，服务应拒绝构造真实用户交易，只允许生成示例或 dry run 教学。

## 开发要点

- 每次更新文档前先核对迁移文档和实际发布对象。
- 代码示例默认 localnet/simulation，主网需要显式配置和版本证明。
- 待完成项写成风险或后续工作，不写成当前能力。

## 检查问题

- 哪些能力可以作为链上已合入基础讲解？
- 哪些能力只能作为本书采用的 GitHub 源码快照或仿真使用？
- 服务初始化缺少版本状态时应如何 fail fast？
