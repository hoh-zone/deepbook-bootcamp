# ch10-05 `oracle.move` 与 `oracle_config.move` 的价格输入

[返回本章](README.md)

## 先看市场问题

读“oracle.move 与 oracle_config.move 的价格输入”时先抓参数边界。Predict 的关键不是某个函数名，而是 oracle、expiry、strike、vault 和 payout 之间的关系。

## 源码入口

- `packages/predict/sources/oracle.move`：`OracleSVI`、spot 更新、basis、SVI、settlement。
- `packages/predict/sources/oracle_config.move`：asset feed id、basis bounds、ask bounds、strike grid 配置。
- `packages/predict/simulations/src/sim.ts`：本地仿真中 oracle refresh 与 mint 合并的交易路径。

## 关键定义

`OracleConfig` 是创建 oracle 前的配置表。它保存 feed id、basis bounds、ask bounds、strike grid 和 staleness 阈值；创建具体 oracle 时，这些值会被快照到 oracle 相关对象中。

```move
public struct OracleConfig has store {
    oracle_grids: Table<ID, OracleGrid>,
    oracle_ask_bounds: Table<ID, AskBounds>,
    asset_basis_bounds: Table<String, BasisBounds>,
    asset_feed_ids: Table<String, u64>,
    spot_staleness_threshold_ms: u64,
    basis_staleness_threshold_ms: u64,
    lazer_authoritative_threshold_ms: u64,
    lazer_settlement_authoritative_threshold_ms: u64,
}

public struct CurvePoint has copy, drop, store {
    strike: u64,
    up_price: u64,
}

public fun new_curve_point(strike: u64, up_price: u64): CurvePoint {
    CurvePoint { strike, up_price }
}
```

`CurvePoint` 是 Predict 定价里非常适合拿来训练 Move 直觉的结构：它很小，但类型边界很硬。strike 和 up price 都是链上整数精度，不是小数；任何前端图表插值都只能用于展示，不能反过来生成未校验的 mint 参数。

## 读市场参数

`oracle.move` 的 `OracleSVI` 维护 underlying asset、Pyth Lazer feed id、expiry、spot、basis、SVI 参数、staleness 时间戳和 settlement price。`update_spot_from_lazer` 面向高频 spot，`update_prices` 面向 operator 推送 spot/forward 并更新 basis，`update_svi` 更新波动率曲面。到期后 oracle 进入 settlement 路径，冻结 settlement price。

`oracle_config.move` 是 Predict 之上的配置层。创建 oracle 前，管理员必须通过 `set_asset_feed_id` 绑定 `asset -> pyth_lazer_feed_id`；可以通过 `set_asset_basis_bounds` 设置 per-asset circuit breaker；创建 oracle 时把 staleness threshold、basis bounds、feed id 和 strike grid 快照到 oracle 或 Predict 配置中。后续修改全局配置不会 retroactively 改老 oracle。

PTB 中不要把 oracle 更新服务当成稳定外部依赖。当前可以在本地仿真里把 refresh 和 mint 放进同一轮 scenario，但生产应用需要先检查迁移状态、operator 权限和 feed 配置，再决定由谁推送价格。

错误解析上，oracle 相关失败通常不是余额问题，而是 feed id 未配置、spot/SVI 超过 staleness threshold、basis 超出 bounds、settlement 已完成或 range key 的 oracle/expiry 与传入 oracle 不匹配。

## Predict 边界判断

- 创建 oracle 前校验 asset、Pyth Lazer feed id、basis bounds 和 strike grid。
- 交易报价必须展示 oracle 更新时间和是否 stale。
- 结算逻辑只引用链上 frozen settlement price，不引用前端缓存价格。

## 动手检查

- `oracle_config` 修改后为什么不能假设老 oracle 自动改变？
- mint 前 oracle stale 和 basis bound 失败应如何提示用户？
- settlement price 冻结后，live redeem 路径为什么不再适用？
