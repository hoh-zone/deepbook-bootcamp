# ch11-08 oracle 更新和结算窗口

[返回本章](README.md)

## 本节目标

- 处理 oracle 更新、staleness 和 settlement 窗口。
- 说明仿真中 oracle refresh 与 mint 合并的交易路径。
- 避免把未完成 Oracle service 写成稳定线上服务。

## 源码关联

- `packages/predict/sources/oracle.move`：spot/SVI 更新和 settlement。
- `packages/predict/sources/oracle_config.move`：feed 和 bounds 配置。
- `packages/predict/simulations/src/sim.ts`：仿真中更新时间和交易窗口。

## 正文

仿真 setup 会先 `set_asset_feed_id("BTC", 1)`，再把 BTC basis bounds 放宽到 10%，因为 CSV 的历史 spot 相邻变化可能超过默认 2%。随后 `updateBasisTx` 调 `oracle::update_prices`，`updateSviTx` 调 `oracle::update_svi`。

生产设计中，Pyth Lazer spot、operator basis 和 SVI 参数是不同频率的数据源。`oracle.move` 有 spot staleness、basis staleness、Lazer authoritative window 和 Lazer settlement authoritative window。前端不能只看最后价格，还要展示数据是否 stale、oracle 是否 active、是否 pending settlement 或 settled。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## 开发要点

- oracle refresh、mint 和 settle 窗口都显示时间戳、staleness threshold 和 operator 来源。
- 仿真可把 refresh + mint 放在 scenario 中，生产文档必须标注 Oracle service 未稳定完成。
- settlement 后禁用 live quote，切换到 settled payout 领取路径。

## 检查问题

- oracle stale 时用户反复签名失败的根因是什么？
- settlement window 内哪些操作应被隐藏或提示高风险？
- 为什么不能把仿真里的 oracle refresh 当作线上服务承诺？
