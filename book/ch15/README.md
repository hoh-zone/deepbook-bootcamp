# ch15 测试、安全、部署与出版级交付

## 本章目标

- 建立 DeepBook 应用的测试体系。
- 梳理金融协议开发的安全清单。
- 给出部署、监控、审计和书稿交付标准。

## 本章学习阶梯

- L2 先建立测试金字塔：Move、SDK、Indexer、前端和 E2E。
- L3 用源码不变量设计测试和安全清单。
- L4 准备部署、升级、监控、故障演练和审计材料。
- L5 把全书项目整理成出版级交付。

## 源码地图

- [packages/deepbook/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/tests)：核心 DeepBook Move 测试。
- [packages/deepbook_margin/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/tests)：Margin 单元和集成测试。
- [packages/predict/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/tests)：Predict 行为测试。
- [crates/indexer/tests/snapshot_tests.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/tests/snapshot_tests.rs)：Indexer 快照测试。
- [docker](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker)：部署容器。

## 小节目录

- [01 测试金字塔](01-testing-pyramid.md)
- [02 Move 单元测试](02-move-unit-tests.md)
- [03 撮合和资金边界测试](03-matching-funds-boundary-tests.md)
- [04 Indexer snapshot tests](04-indexer-snapshot-tests.md)
- [05 SDK 测试](05-sdk-tests.md)
- [06 前端端到端测试](06-frontend-e2e-tests.md)
- [07 金融协议安全清单](07-protocol-security-checklist.md)
- [08 对象权限风险](08-object-permission-risks.md)
- [09 版本升级](09-version-upgrades.md)
- [10 部署拓扑](10-deployment-topology.md)
- [11 日志、指标和 SLO](11-logs-metrics-slo.md)
- [12 故障演练](12-incident-drills.md)
- [13 审计准备](13-audit-preparation.md)
- [14 出版级书稿标准](14-publishing-quality-standard.md)
- [15 每章验收标准](15-chapter-acceptance-standard.md)
- [16 全书项目 README](16-project-readme.md)
- [17 代码运行手册](17-code-runbook.md)
- [18 附录](18-appendices.md)

## 本章代码

- `code/s01-move-test-template/`：Move 测试模板。
- `code/s02-sdk-test-template/`：SDK 测试模板。
- `code/s03-indexer-snapshot-template/`：Indexer 快照测试模板。
- `code/s04-e2e-test-template/`：端到端测试模板。
- `code/s05-deployment-compose/`：部署 compose 模板。
- `code/s06-audit-checklist/`：审计清单。

## Move 高阶穿插点

- 测试要覆盖 Move 单元测试、PTB dry run、SDK 参数测试、Indexer 重放和生产监控，不要只测 happy path。
- 安全审查从资源开始：谁能创建、转移、借用、销毁、升级和暂停对象。
- 出版级交付要求每个结论都有源码、测试或交易证据，不能只写经验判断。

## 常见错误

- 只测成功路径。
- 把 dry run 当作完整安全测试。
- 忘记 Indexer 和 Server 的版本兼容。
- 没有记录交易 digest，导致线上问题无法追踪。
- 书稿代码无法运行或缺少依赖说明。

## 本章检查清单

- [ ] 每类交易都有成功和失败测试。
- [ ] 每个 API 有分页、限流和错误响应。
- [ ] 每个生产服务有健康检查。
- [ ] 每个章节代码示例有 README。
- [ ] 全书术语和路径已统一。

## 进阶练习

- 为 Spot 交易写一份测试矩阵。
- 为 Margin 清算写一份威胁模型。
- 为 Indexer 设计一次全量重放演练。


