# ch10-01 版本状态、网络状态和产品边界

[返回本章](README.md)

## 先看市场问题

读“版本状态、网络状态和产品边界”时先抓参数边界。Predict 的关键不是某个函数名，而是 oracle、expiry、strike、vault 和 payout 之间的关系。

## 源码入口

- [PREDICT_MIGRATION.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/PREDICT_MIGRATION.md)：PR 1/2 合入状态与后续阶段的计划状态。
- `packages/predict/sources/oracle.move`、`vault/vault.move`、`config/pricing_config.move`、`config/risk_config.move`：迁移文档中已合入能力的核心文件。
- `packages/predict/sources/predict.move`、`registry.move`、`predict_manager.move`、`market_key/range_key.move`：本书采用的 GitHub 源码快照中可读到的后续模块，生产状态需要另行核对。

## 读市场参数

Predict 是 DeepBook 生态里的预测市场/二元期权协议，不是 DeepBookV3 spot order book，也不是 DeepBook Margin 借贷池。它复用 DeepBook 的 `BalanceManager` 账户模型，但资金池、oracle、PLP 和结算逻辑都在 `packages/predict` 内部实现。

从迁移计划看，稳定事实是 PR 1 与 PR 2：`oracle.move`、`math.move`、`constants.move`、`vault.move`、`pricing_config.move`、`risk_config.move` 已列为 merged。PR 3 到 PR 11 仍标记为 next、blocked 或未开始。GitHub 源码已经出现更多模块，适合源码精读和本地仿真，但生产应用必须按实际部署包、审计状态和网络公告再确认，不能假设 Predict Indexer、Predict Server、部署脚本和 Oracle 服务已经主网稳定。

文档写作时建议把 Predict 标为“本书采用的 GitHub 源码快照 + 迁移中协议”。如果要在应用里展示网络状态，应把 package ID、registry ID、predict object ID、oracle ID 和发布网络都放进配置，并附带 `migrationStatus` 或 `releaseChannel` 字段，避免 UI 把示例对象误读为主网。

本章后续小节提到的 Indexer、Server、Oracle service 都只作为需求或未来读模型讨论。可靠的链上事实来自 Move 对象、事件和 dry run 结果；链下服务在迁移文档未完成前不能写成稳定依赖。

## Predict 边界判断

- 每个 Predict 说明都先标注来源是 `PREDICT_MIGRATION.md` 还是本书采用的 GitHub 源码快照。
- 写主网能力时必须有实际 package/object ID 和部署状态；没有时只写本地仿真或接口设计。
- 把 Spot、Margin、Predict 的资金池分开描述，不把 Predict vault 写成 DeepBook pool。

## 动手检查

- 哪些模块在迁移文档中明确 merged，哪些只是GitHub 源码可见？
- 如果前端配置里没有 Predict package ID 和版本状态，应该阻止哪些交易入口？
- 为什么不能把 Predict Indexer/Server 写成已经上线的稳定能力？
