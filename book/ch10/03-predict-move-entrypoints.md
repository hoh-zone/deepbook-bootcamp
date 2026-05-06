# ch10-03 `predict.move` 的对外入口

[返回本章](README.md)

## 本节目标

- 逐条读懂 `predict.move` 中交易者和 LP 可直接调用的 Move 入口。
- 梳理 mint、live redeem、settled redeem、supply、withdraw 的资金和状态变化。
- 明确 `Predict` 共享对象中配置、vault、fee reserve 和 limiter 的职责。

## 源码关联

- `packages/predict/sources/predict.move`：`Predict` 对象、`mint`、`redeem`、`redeem_permissionless`、`supply`、`withdraw`。
- `packages/predict/sources/predict_manager.move`：manager owner 校验、余额 withdraw、头寸增减。
- `packages/predict/sources/accounting/fee_reserve.move`、`vault/vault.move`、`vault/plp.move`：费用、vault 资产和 PLP 铸烧。

## 源码定义

Predict 的主对象定义：

```move
public struct Predict has key {
    id: UID,
    vault: Vault,
    fee_reserve: FeeReserve,
    treasury_cap: TreasuryCap<PLP>,
    pricing_config: PricingConfig,
    risk_config: RiskConfig,
    treasury_config: TreasuryConfig,
    oracle_config: OracleConfig,
    withdrawal_limiter: RateLimiter,
    trading_paused: bool,
}
```

两个最重要的交易入口：

```move
public fun mint<Quote>(
    predict: &mut Predict,
    manager: &mut PredictManager,
    oracle: &OracleSVI,
    key: RangeKey,
    quantity: u64,
    clock: &Clock,
    ctx: &mut TxContext,
)

public fun redeem<Quote>(
    predict: &mut Predict,
    manager: &mut PredictManager,
    oracle: &OracleSVI,
    key: RangeKey,
    quantity: u64,
    clock: &Clock,
    ctx: &mut TxContext,
)
```

这两个签名解释了 Predict 应用为什么比 Spot 应用更依赖上下文：交易不仅需要 manager 和共享对象，还需要 oracle、range key、clock 和 quote 类型。错误也不只来自余额不足，还可能来自交易暂停、oracle stale、区间非法、ask price 越界或 vault 风险限制。

## 正文

`predict.move` 的核心共享对象是 `Predict`，字段包括 `vault`、`fee_reserve`、`treasury_cap<PLP>`、`pricing_config`、`risk_config`、`treasury_config`、`oracle_config`、`withdrawal_limiter` 和 `trading_paused`。

交易入口 `mint<Quote>(predict, manager, oracle, key, quantity, clock, ctx)` 要求 `ctx.sender() == manager.owner()`，调用 `mint_internal`。内部先校验 quote asset、trading pause、oracle 和 range key，再通过 `quote_mint_amounts` 得到 principal 与 fee，从 `PredictManager` withdraw quote，fee 进入 `FeeReserve`，LP share 回流 vault，剩余 payment 进入 vault，最后 `manager.increase_position` 并触发 `PositionMinted`。

`redeem<Quote>` 根据 `oracle.is_settled()` 分支。未结算时走 `redeem_live_internal`，用户卖回当前 fair value 并支付 live redeem fee；已结算时走 `redeem_settled_internal`，按 `RangeKey::settled_payout` 支付，结算 redeem 不收费。`redeem_permissionless` 允许任何执行者把已结算头寸卖入 manager 余额，适合 keeper 或自动领取。

LP 入口是 `supply<Quote>` 和 `withdraw<Quote>`。`supply` 接收 quote coin，刷新 LP 相关 MTM，按 `amount * total_plp / vault_value` 计算 shares；首次供应时按 amount mint PLP。`withdraw` 先校验 MTM freshness，再按 shares 占比计算 amount，消耗 withdrawal limiter，烧毁 PLP 并从 vault 支付 quote。

应用层封装 `mint` 的 PTB 顺序应先保证 manager 已存在并有 quote 余额，然后传入 `predict`、`manager`、`oracle`、`RangeKey`、`quantity`、`Clock`。错误解析应优先判断 sender 是否等于 manager owner、quote asset 是否在 treasury whitelist、oracle 是否 stale 或已 settle、ask price 是否越界。

`redeem_permissionless` 是自动化领取的基础，但它不代表存在稳定 keeper 服务。文档和 SDK 可以设计 keeper 调用接口，却必须标注当前迁移状态，不能把它描述成已经上线的 Predict Server 能力。

## 开发要点

- 把每个入口的输入对象、type argument、sender 约束和事件分开列清楚。
- mint 说明必须覆盖 manager withdraw、fee split、vault accept payment、position increase。
- withdraw 说明必须包含 MTM freshness、rate limiter、PLP burn 和 quote payout。

## 检查问题

- `mint_internal` 失败最常见会落在 owner、quote asset、oracle、range key 还是 ask bounds？
- 为什么 settled redeem 是 zero-fee，而 live redeem 需要 fee？
- `redeem_permissionless` 适合什么自动化场景，为什么不能据此假设稳定 Server 已存在？
