# ch00-10 DeepBook Predict 是什么

[返回本章](README.md)

DeepBook Predict 是预测市场方向的独立产品包，源码位于 [packages/predict/sources](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources)。它和现货订单簿的交易形态不同，不是简单买卖 base/quote，而是围绕某个 oracle、到期时间和价格区间生成预测头寸。

官方文档当前把 Predict 描述为 Testnet integration surface，而不是 Mainnet 稳定产品面。书中所有对象 ID、package ID、public server、entry points 和网络状态都必须带这个前提。预测市场章节不能只讲“源码里有什么”，还要讲“哪些能力可以按官方 Testnet 集成目标使用，哪些需要等部署和文档更新确认”。

主入口是 [predict.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/predict.move)。源码注释说明该模块协调 vault、oracle config、manager 和 pricing 层，负责公开交易和 LP flows。事件包括 `PositionMinted`、`PositionRedeemed`、`Supplied`、`Withdrawn`、配置更新和 oracle 边界更新等。

Predict 的核心模型可以拆成五部分。第一，`PredictManager` 表示用户的预测头寸管理入口。第二，`Vault` 保存 LP 资金、头寸资产和风险暴露。第三，`OracleConfig` 和 `oracle` 相关模块提供价格、区间和结算依据。第四，`PricingConfig`、`RiskConfig` 和 `TreasuryConfig` 控制费用、风险和金库参数。第五，`FeeReserve` 处理 LP、协议和保险费用分配。

这类产品对数据层要求很高。前端不只要展示“买入/卖出”，还要展示到期时间、strike 区间、oracle 状态、费用、vault 可用资金、最大赔付、LP 份额和结算状态。应用开发时，需要同时理解链上对象和索引数据，否则用户很难判断头寸价值。

官方集成模型建议组合三类数据：public Predict server 用于页面和历史数据，checkpoint/event streaming 用于低延迟 oracle 更新，关键钱包流程前后再直接读取链上对象。这个三层模型会贯穿 ch10、ch11、ch12 和 ch13。

本书会把 Predict 放在较后章节。原因是它需要读者先掌握 Move 对象、DeepBook 现货基础、SDK 交易构造、Indexer 和风控展示。直接从 Predict 开始会缺少必要上下文。

官方入口：

- [DeepBook Predict overview](https://docs.sui.io/onchain-finance/deepbook-predict/)
- [DeepBook Predict design](https://docs.sui.io/onchain-finance/deepbook-predict/design)
- [DeepBook Predict contract information](https://docs.sui.io/onchain-finance/deepbook-predict/contract-information)

## 本节检查

- [ ] 能说明 Predict 和现货订单簿的产品差异。
- [ ] 能列出 Predict 的 manager、vault、oracle、pricing、fee reserve 五个核心区域。
- [ ] 能解释为什么 Predict 前端强依赖数据层。
- [ ] 能说明为什么 Predict 章节必须标注 Testnet / Mainnet 边界。
