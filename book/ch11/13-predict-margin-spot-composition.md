# ch11-13 Predict 与 Margin/Spot 组合产品的设计边界

[返回本章](README.md)

## 本节目标

- 划定 Predict 与 Margin/Spot 组合产品的工程边界。
- 说明跨产品 PTB 组合时的对象、风险和价格源前置条件。
- 避免把 Predict vault、MarginPool 和 DeepBook pool 混为一谈。

## 源码关联

- `packages/predict/sources/predict.move`：Predict 交易入口。
- `packages/deepbook/sources/pool.move`：Spot 下单/swap 入口。
- `packages/deepbook_margin/sources/margin_manager.move`：Margin 用户入口。

## 正文

Spot 是现货撮合和 swap，Margin 是借贷、杠杆和清算，Predict 是到期二元/区间赔付。组合产品可以在应用层把 spot hedge、margin borrow 和 predict position 放在一个投资组合视图里，但不要把三者的风险混成一个合约能力。

例如“用 Margin 借 USDC 买 Predict UP”是应用层 PTB 或多步流程，不代表 Predict vault 接受 MarginPool 债权作为抵押。当前 Predict 的 quote 资产白名单由 `treasury_config` 控制，头寸由 `PredictManager` 管理。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## 开发要点

- 组合产品先定义资金来源：Spot BalanceManager、MarginManager、PredictManager 不混用。
- 跨产品 PTB 需要分别校验 oracle、risk ratio、vault exposure 和 object version。
- 不把 Predict payout 当作 Margin 抵押品，除非协议明确支持。

## 检查问题

- Predict vault、MarginPool、DeepBook pool 的职责差异是什么？
- 一个组合 PTB 同时触碰 Spot/Margin/Predict 时，失败面会增加哪些？
- 哪些组合只能作为产品设计，不能直接写成当前协议能力？
