# ch11-02 创建 Predict market

[返回本章](README.md)

## 先跑通场景

这一节先把场景落到可执行流程：读者需要看到对象从哪里来，PTB 如何构造，交易前检查什么，失败后如何回到源码定位。

## 源码入口

- `packages/predict/sources/registry.move`：创建 Predict、Oracle、PredictManager 和配置入口。
- `packages/predict/sources/oracle_config.move`：asset feed、basis bounds 和 strike grid。
- `packages/predict/simulations/src/sim.ts`：localnet setup 中创建或加载对象的流程。

## 从仿真到交易

源码里 market 不是一个独立 `Market` 对象，而是 `OracleSVI + OracleGrid + RangeKey` 的组合。管理员或 oracle operator 的创建路径在 `registry.move`：

1. `create_predict<Quote>(registry, admin_cap, currency, plp_treasury_cap, clock, ctx)` 创建单例 `Predict`。
2. `enable_quote_asset<Quote>(predict, admin_cap, currency)` 允许某个 6 位 quote asset 进入 vault。
3. `create_oracle_cap(admin_cap, ctx)` 给 operator 一个 `OracleSVICap`。
4. `set_asset_feed_id(predict, admin_cap, "BTC", feed_id)` 先绑定 Pyth Lazer feed。
5. `set_asset_basis_bounds(...)` 可选，设置 per-asset basis circuit breaker。
6. `create_oracle(registry, predict, cap, underlying_asset, expiry, min_strike, tick_size, clock, ctx)` 创建 oracle 并在 Predict 中初始化 strike grid。
7. `oracle::activate(oracle, cap, clock)` 后才能报价。

应用创建市场前要检查 expiry 仍在未来、feed id 已配置、strike grid 满足 min strike 和 tick size 约束、basis bounds 不会让真实行情频繁触发 circuit breaker。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## Predict 应用判断

- create market PTB 按 registry、oracle config、oracle 创建、predict 激活的顺序组织。
- 管理员 cap、operator 权限、feed id、basis bounds 和 strike grid 都从配置读取。
- 本节只描述 localnet/setup 或当前发布入口，不写成稳定部署脚本。

## 动手检查

- 创建 market 前必须准备哪些 cap 和配置对象？
- feed id 或 basis bounds 缺失时 dry run 会暴露哪类问题？
- 为什么 create market 示例不能直接等同于主网部署流程？
