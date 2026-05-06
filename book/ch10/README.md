# ch10 DeepBook Predict 协议源码精读

## 本章目标

读完本章，你应该能从源码层面解释 DeepBook Predict 的核心对象、交易入口、资金路径和风险边界，并能判断哪些能力已经存在于本书采用的 GitHub 源码快照，哪些仍只是迁移计划。

本章使用的源码快照位于 [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)。需要特别注意版本状态：[PREDICT_MIGRATION.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/PREDICT_MIGRATION.md) 写明 PR 1 Oracle 与 PR 2 Vault + Configs 已合入 main，PR 3 Market Key + Predict Manager、PR 4 Predict Core + Registry、PR 5 测试以及后续 Indexer/Server/脚本服务仍处于计划或阻塞状态。源码快照中已经可以看到 `predict.move`、`registry.move`、`predict_manager.move`、`market_key/range_key.move` 和仿真代码，但书中不能把迁移文档中未完成的 Indexer、Server、部署脚本、Oracle 服务写成稳定主网能力。

## 本章学习阶梯

- L2 先理解 Predict 的产品模型：区间、到期、oracle、LP vault。
- L3 读 `predict.move`、manager、vault、pricing、risk 和 fee reserve。
- L4 能解释 mint、redeem、settle、withdraw 的状态变化。
- L5 能判断 Predict 当前网络/版本边界和可产品化范围。

## 源码地图

- [packages/predict/sources/predict.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/predict.move)：Predict 共享对象、交易入口、LP 入口、配置读写和事件。
- [packages/predict/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/registry.move)：`Registry`、`AdminCap`、创建 Predict、创建 Oracle、创建 PredictManager、配置管理入口。
- [packages/predict/sources/predict_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/predict_manager.move)：用户资金账户和头寸表，内部包裹 DeepBook `BalanceManager`。
- [packages/predict/sources/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/oracle.move)：SVI oracle、Pyth Lazer spot、basis、结算价格和二元定价输入。
- [packages/predict/sources/oracle_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/oracle_config.move)：strike grid、ask bounds、basis bounds、asset -> Pyth Lazer feed id。
- [packages/predict/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/vault/vault.move)：LP vault 余额、区间敞口、MTM 和最大赔付。
- [packages/predict/sources/vault/plp.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/vault/plp.move)：Predict LP token，6 位精度。
- [packages/predict/sources/market_key/range_key.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/market_key/range_key.move)：市场 key，表示 `(oracle, expiry, lower, higher]`。
- [packages/predict/sources/config/pricing_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/config/pricing_config.move)：base fee、min fee、utilization multiplier、ask price bound。
- [packages/predict/sources/config/risk_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/config/risk_config.move)：总敞口百分比和 MTM 新鲜度。
- [packages/predict/sources/config/treasury_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/config/treasury_config.move)：quote asset 白名单和 6 位精度校验。
- [packages/predict/sources/accounting/fee_reserve.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/accounting/fee_reserve.move)：LP、protocol、insurance fee reserve 拆分。
- [packages/predict/sources/helper/strike_matrix.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/helper/strike_matrix.move)：固定 strike 网格下的区间库存、MTM 和 worst-case payout。
- [packages/predict/sources/helper/rate_limiter.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/helper/rate_limiter.move)：LP 提款 token bucket 限速器。
- [packages/predict/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/tests)：当前测试覆盖 fee reserve、pricing config、rate limiter、oracle cap，迁移文档计划中的端到端 Predict 测试尚不能视为已完成。

## 小节目录

- [01 版本状态、网络状态和产品边界](01-version-network-boundaries.md)
- [02 预测市场、二元期权、区间市场和 LP vault](02-prediction-markets-binary-range-lp.md)
- [03 `predict.move` 的对外入口](03-predict-move-entrypoints.md)
- [04 `predict_manager.move` 的用户账户和头寸管理](04-predict-manager.md)
- [05 `oracle.move` 与 `oracle_config.move` 的价格输入](05-oracle-and-oracle-config.md)
- [06 `vault/vault.move` 的资金池、mint 和 settle](06-vault-mint-settle.md)
- [07 `vault/plp.move` 的 LP 份额和收益](07-plp-shares.md)
- [08 `range_key.move` 的 market key](08-range-key-market-key.md)
- [09 pricing config、risk config、treasury config](09-pricing-risk-treasury-configs.md)
- [10 strike matrix、rate limiter 和 math helper](10-strike-matrix-rate-limiter-math.md)
- [11 fee reserve 和协议收费路径](11-fee-reserve.md)
- [12 Predict 事件和未来 Indexer 需求](12-predict-events-future-indexer.md)
- [13 结合测试分析 mint、exercise、settle、withdraw](13-tests-mint-exercise-settle-withdraw.md)
- [14 已完成和待完成模块](14-completed-and-pending-modules.md)

## 本章代码

- `book/ch10/code/s01-predict-source-map/`：生成 Predict 模块依赖图。
- `book/ch10/code/s02-market-key-builder/`：构造 `RangeKey` 所需参数。
- `book/ch10/code/s03-pricing-calculator/`：实现简化 fee 计算器。
- `book/ch10/code/s04-vault-accounting/`：模拟 Predict vault、PLP、fee reserve 和 payout 的账本变化。

## Move 高阶穿插点

- Predict 是学习 Move 定价型协议的案例：头寸不是普通订单，而是区间、到期、oracle 和 vault 风险的组合。
- 读配置模块时区分全局参数、单 oracle override、资产级边界和管理员权限。
- 事件不仅服务历史展示，也服务用户理解费用、赎回、结算和 LP 份额变化。

## 常见错误

- 把 `PREDICT_MIGRATION.md` 中 Phase 2 的 Indexer/Server 写成已上线能力。
- 把 quote fee 当成 bps；源码中它是每单位绝对价格增量。
- 只用 strike 和方向当市场 ID，忽略 oracle ID 与 expiry。
- LP 页面只展示 vault balance，不展示 MTM、max payout 和 withdrawal limiter。
- 把 settled redeem 写成收费交易；源码中 settled redeem 是 zero-fee。
- 把 Predict vault 误写成 DeepBookV3 spot pool 或 MarginPool。

## 本章检查清单

- 能解释 `Predict`、`Registry`、`PredictManager`、`OracleSVI`、`Vault`、`PLP` 的对象关系。
- 能写出 mint 的资金路径：manager withdraw、fee split、vault accept payment、position increase。
- 能解释 live redeem 与 settled redeem 的区别。
- 能说明 `RangeKey` 为什么必须包含 oracle 和 expiry。
- 能说明 pricing config 与 risk config 分别控制什么。
- 能区分本书采用的 GitHub 源码快照能力和迁移计划未完成事项。

## 进阶练习

1. 设计一个 SUI 价格 UP/DOWN 市场，写出 oracle asset、feed id、expiry、strike grid、两个 `RangeKey`。
2. 给定 vault balance、total MTM、max payout 和 fee 参数，计算 LP 当前 NAV 与用户 mint 的 all-in cost。
3. 为 Predict 设计最小事件索引表，至少覆盖 market、oracle update、position、LP share 和 fee accrual。

