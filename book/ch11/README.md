# ch11 DeepBook Predict 应用开发与仿真

## 本章目标

本章把 ch10 的源码对象翻译成应用开发路径：如何组织市场页、如何创建 market、如何创建或加载 `PredictManager`、如何构造 mint/redeem/supply/withdraw PTB，以及如何使用本地 simulation runtime 评估 gas、延迟和 LP 风险。

版本状态必须先说清楚：本章参考的是 [packages/predict/simulations](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/simulations) 与 GitHub 源码快照中的 `packages/predict/sources/*`。[PREDICT_MIGRATION.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/PREDICT_MIGRATION.md) 中 Predict Indexer、Predict Server、部署脚本和 oracle services 尚未完成；因此本章示例是本地开发和仿真骨架，不代表稳定主网 SDK 或稳定 API。

## 本章学习阶梯

- L1 先设计 Predict 页面需要展示的信息。
- L2 构造 mint、redeem、LP supply/withdraw 的交易或仿真路径。
- L3 把仿真结果映射到 vault、oracle、pricing 和 risk 源码。
- L4 做出预测市场 CLI 或 LP 收益曲线。

## 源码地图

- [packages/predict/simulations/run.sh](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/simulations/run.sh)：创建或恢复 localnet simulation instance，发布包，运行 setup/sim/analyze。
- [packages/predict/simulations/src/runtime.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/simulations/src/runtime.ts)：Sui client、PTB 构造、对象 ID 派生、执行重试和 gas 汇总。
- [packages/predict/simulations/src/sim.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/simulations/src/sim.ts)：setup 流程、scenario 执行、oracle refresh + mint 合并交易、结果汇总。
- [packages/predict/simulations/src/shared.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/simulations/src/shared.ts)：scenario CSV 解析、results schema、state/results 路径。
- [packages/predict/simulations/visualize.py](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/simulations/visualize.py)：把 `results.json` 渲染为图表。
- [packages/predict/simulations/data/scenario_mar6_1000mints.csv](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/simulations/data/scenario_mar6_1000mints.csv)：示例价格、SVI 和 mint 场景。

## 小节目录

- [01 Predict 应用的信息架构](01-predict-app-information-architecture.md)
- [02 创建 Predict market](02-create-predict-market.md)
- [03 创建或加载 PredictManager](03-create-load-predict-manager.md)
- [04 deposit quote 资产](04-deposit-quote-asset.md)
- [05 mint binary position 的交易构造](05-mint-binary-position.md)
- [06 mint range position 的交易构造](06-mint-range-position.md)
- [07 LP supply 和 withdraw](07-lp-supply-withdraw.md)
- [08 oracle 更新和结算窗口](08-oracle-updates-settlement-window.md)
- [09 settled market 的收益领取](09-redeem-settled-market.md)
- [10 前端展示风险和收益](10-frontend-risk-reward-display.md)
- [11 仿真脚本如何构造市场路径](11-simulation-market-path.md)
- [12 仿真结果如何可视化](12-visualize-simulation-results.md)
- [13 Predict 与 Margin/Spot 组合产品的设计边界](13-predict-margin-spot-composition.md)
- [14 本章实战：预测市场 CLI](14-predict-market-cli.md)
- [15 本章实战：仿真价格路径和 LP 收益曲线](15-simulation-lp-curve.md)

## 本章代码

- `book/ch11/code/s01-create-predict-market/`：市场创建 PTB 顺序。
- `book/ch11/code/s02-mint-position/`：用户创建 manager、deposit、mint position。
- `book/ch11/code/s03-supply-lp/`：LP supply 与 withdraw 的交易顺序。
- `book/ch11/code/s04-settle-market/`：oracle settlement 后的 settle 和 redeem 流程。
- `book/ch11/code/s05-simulation-report/`：读取 `results.json` 生成报告。

## Move 高阶穿插点

- 仿真不是前端假数据，而是把 Move 中的 pricing、risk、vault 约束搬到链下预演。
- Predict UI 要把资源状态翻译成人能理解的风险语言，但不能丢掉链上原始整数和 oracle 时间。
- 交易前检查要覆盖暂停、quote asset 是否启用、oracle 是否 stale、vault 是否有足够流动性。

## 常见错误

- 直接调用 `predict::mint`，但用户没有创建或充值 `PredictManager`。
- 前端把 fair price 当最终成交价，漏掉 fee 和 ask bounds。
- 在 UI 中只显示 “UP/DOWN”，没有显示 expiry、oracle ID 和 settlement rule。
- 把 localnet DUSDC mint 示例复制到生产环境。
- 忽略 oracle staleness，导致用户在不可报价状态下反复签名失败。
- 把 simulation localnet 结果写成主网性能承诺。

## 本章检查清单

- 能从 owner 派生 PredictManager ID，并在不存在时创建。
- 能构造 binary 和 range `RangeKey`。
- 能描述 create market 的 admin/operator 前置条件。
- 能解释 `run.sh` 的 setup、sim、resume、analyze 阶段。
- 能读懂 `results_v2` 的 gas 和 wallMs 字段。
- 能在 UI 上区分交易者风险和 LP 风险。

## 进阶练习

1. 给 `sim.ts` 增加 redeem action，并在 `results.json` 中增加 redeem summary。
2. 为预测市场 CLI 设计 abort code 映射表。
3. 用一组自定义 CSV 比较不同 basis bounds 对仿真成功率和 gas 的影响。

