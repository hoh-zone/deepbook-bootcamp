# ch08-06 风险率

[返回本章](README.md)

## 先看风险边界

这里先把问题放到风险控制面里。“风险率”不是在 DeepBook 旁边加一层 UI，而是把 registry、manager、oracle、borrow/supply state 接进同一条链上风控路径。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/helper/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/oracle.move)
- [packages/deepbook_margin/sources/margin_registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_registry.move)
- `book/ch08/code/s02-risk-ratio-calculator/README.md`

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 关键定义

风险阈值来自 `PoolConfig` 和 `RiskRatios`：

```move
public struct PoolConfig has copy, drop, store {
    base_margin_pool_id: ID,
    quote_margin_pool_id: ID,
    risk_ratios: RiskRatios,
    user_liquidation_reward: u64,
    pool_liquidation_reward: u64,
    enabled: bool,
    extra_fields: VecMap<String, u64>,
}

public struct RiskRatios has copy, drop, store {
    min_withdraw_risk_ratio: u64,
    min_borrow_risk_ratio: u64,
    liquidation_risk_ratio: u64,
    target_liquidation_risk_ratio: u64,
}
```

这不是前端配置项，而是链上风控参数。`enabled` 控制某个 DeepBook pool 是否允许 Margin 交易；`base_margin_pool_id` 和 `quote_margin_pool_id` 指向可借资产池；四个 risk ratio 决定借款、提款、清算触发和清算目标。UI 中如果只显示一个“健康度”，会掩盖这些动作边界。

## 读风险控制面

风险率在 `margin_manager.move` 的 `risk_ratio`、`risk_ratio_unsafe`、`risk_ratio_int` 中计算。直观公式是：

```text
risk_ratio = assets_in_debt_unit / debt
```

如果 debt 是 base，则把 quote 资产按 oracle 价格换算成 base；如果 debt 是 quote，则把 base 资产换算成 quote。`assets_in_debt_unit` 会包含 manager 中可用资产、结算后资产和订单簿中相关余额。

四个阈值决定动作边界：

- `min_borrow_risk_ratio`：借款后必须高于该值，否则 `borrow_base` / `borrow_quote` 抛 `EBorrowRiskRatioExceeded`。
- `min_withdraw_risk_ratio`：提款后必须高于该值，否则 `withdraw` 抛 `EWithdrawRiskRatioExceeded`。
- `liquidation_risk_ratio`：低于该值后 `can_liquidate` 为真。
- `target_liquidation_risk_ratio`：清算时用于计算最多应还多少债，目标是把剩余仓位拉回目标风险率附近。

生产应用不要只用前端本地价格判断风险率。交易前必须用当前 Pyth price object 做 dry run 或 dev inspect，因为链上还会检查 stale price、置信区间和 EWMA 偏差。

## 工程旁白

风险率计算要先确定 debt asset。借 quote 开多时，base 资产需要按价格换算成 quote；借 base 开空时，quote 资产需要换算成 base。价格方向搞反会直接导致健康度展示错误。

风险率不是只看 `BalanceManager` 可用余额，还要考虑 DeepBook Pool 中已结算但未提取的金额、订单相关余额以及随时间增长的 debt amount。清算机器人应使用链上读接口或 dev inspect，避免 indexer 延迟造成误判。

## 风控判断

- 显示风险率时同时展示 debt asset、债务金额、阈值和价格时间戳。
- 借款、提款、交易、TPSL 创建前都要模拟操作后的风险率。
- 把风险率小于 1、低于清算阈值、低于提款阈值分别映射成不同 UI 状态。

## 动手检查

- 当前风险率是用 base 计价还是 quote 计价？
- 哪些资产余额被计入 `assets_in_debt_unit`，open orders 是否已处理？
- 价格过期或 confidence 过大时，应用是阻止交易还是只给提示？
