# ch10-12 Predict 事件和未来 Indexer 需求

[返回本章](README.md)

## 本节目标

- 识别 Predict 事件可支持的读模型，以及当前 Indexer 仍是未来需求。
- 设计 market、oracle、position、LP share、fee accrual 的最小索引字段。
- 避免把事件索引需求写成已经稳定上线的 Server API。

## 源码关联

- `packages/predict/sources/predict.move`：mint、redeem、supply、withdraw、配置相关事件。
- `packages/predict/sources/oracle.move`：spot/SVI/settlement 更新事件。
- [PREDICT_MIGRATION.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/PREDICT_MIGRATION.md)：Indexer/Server 仍处计划或阻塞状态的版本依据。

## 正文

当前合约事件覆盖交易、LP、配置和 oracle 变化：`PositionMinted`、`PositionRedeemed`、`Supplied`、`Withdrawn`、`TradingPauseUpdated`、`PricingConfigUpdated`、`RiskConfigUpdated`、`OracleAskBoundsSet`、`OracleAskBoundsCleared`、`QuoteAssetEnabled`、`QuoteAssetDisabled`、`OracleFeedIdSet`、`OracleBasisBoundsUpdated`、`OracleCreated`、`OracleActivated`、`OraclePricesUpdated`、`OracleSVIUpdated`、`OracleSettled`、`FeeAccrued` 等。

但 `PREDICT_MIGRATION.md` 把 Predict schema、indexer 和 server 放在 Phase 2，尚未完成。因此本章只能说事件具备未来 Indexer 的数据基础，不能说已有稳定 Predict API。应用如果现在基于源码快照开发，需要直接读对象和交易事件，或自建临时索引。

事件可以支持两类读模型：用户账本和市场风险。用户账本关注 manager、owner、range key、quantity、cost、payout、fee；市场风险关注 oracle、expiry、vault balance、MTM、max payout、PLP supply 和 limiter 状态。

在 Indexer 未完成前，应用不能依赖公共 Predict API 返回完整 PnL。可行的过渡方案是保存用户签名交易的 digest 与 dry run 摘要，再用 RPC 查询交易详情重建局部历史。

## 开发要点

- 所有 Indexer 描述都用“未来需求”“可设计为”，不写成已上线能力。
- 事件 schema 必须包含 package/network/oracle/expiry/lower/higher。
- 前端缓存要能在 RPC 交易查询失败时显示“历史待同步”，不能覆盖链上余额。

## 检查问题

- Position 事件和 Vault/PLP 事件分别服务哪类页面？
- 没有稳定 Indexer 时，用户 PnL 可以从哪些数据源临时恢复？
- 未来 Server API 和链上对象状态发生冲突时，应以哪个为准？
