# ch10-05 `oracle.move` 与 `oracle_config.move` 的价格输入

[返回本章](README.md)

## 本节目标

- 理解 Predict oracle 如何把 spot、basis、SVI 曲面和 settlement price 输入定价。
- 区分 `oracle.move` 的价格状态和 `oracle_config.move` 的管理员配置快照。
- 识别 oracle stale、basis bounds、feed id 缺失对 mint/redeem 的影响。

## 源码关联

- `packages/predict/sources/oracle.move`：`OracleSVI`、spot 更新、basis、SVI、settlement。
- `packages/predict/sources/oracle_config.move`：asset feed id、basis bounds、ask bounds、strike grid 配置。
- `packages/predict/simulations/src/sim.ts`：本地仿真中 oracle refresh 与 mint 合并的交易路径。

## 正文

`oracle.move` 的 `OracleSVI` 维护 underlying asset、Pyth Lazer feed id、expiry、spot、basis、SVI 参数、staleness 时间戳和 settlement price。`update_spot_from_lazer` 面向高频 spot，`update_prices` 面向 operator 推送 spot/forward 并更新 basis，`update_svi` 更新波动率曲面。到期后 oracle 进入 settlement 路径，冻结 settlement price。

`oracle_config.move` 是 Predict 之上的配置层。创建 oracle 前，管理员必须通过 `set_asset_feed_id` 绑定 `asset -> pyth_lazer_feed_id`；可以通过 `set_asset_basis_bounds` 设置 per-asset circuit breaker；创建 oracle 时把 staleness threshold、basis bounds、feed id 和 strike grid 快照到 oracle 或 Predict 配置中。后续修改全局配置不会 retroactively 改老 oracle。

PTB 中不要把 oracle 更新服务当成稳定外部依赖。当前可以在本地仿真里把 refresh 和 mint 放进同一轮 scenario，但生产应用需要先检查迁移状态、operator 权限和 feed 配置，再决定由谁推送价格。

错误解析上，oracle 相关失败通常不是余额问题，而是 feed id 未配置、spot/SVI 超过 staleness threshold、basis 超出 bounds、settlement 已完成或 range key 的 oracle/expiry 与传入 oracle 不匹配。

## 开发要点

- 创建 oracle 前校验 asset、Pyth Lazer feed id、basis bounds 和 strike grid。
- 交易报价必须展示 oracle 更新时间和是否 stale。
- 结算逻辑只引用链上 frozen settlement price，不引用前端缓存价格。

## 检查问题

- `oracle_config` 修改后为什么不能假设老 oracle 自动改变？
- mint 前 oracle stale 和 basis bound 失败应如何提示用户？
- settlement price 冻结后，live redeem 路径为什么不再适用？
