# ch08-06 风险率

[返回本章](README.md)

## 本节目标

- 掌握 Margin 风险率的计算口径：以 debt asset 为单位折算资产并除以债务。
- 能解释四个风险率阈值如何分别控制借款、提款、清算触发和清算目标。
- 能说明为什么风险率必须依赖链上 oracle 检查，而不能只信前端价格。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/helper/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/oracle.move)
- [packages/deepbook_margin/sources/margin_registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_registry.move)
- `book/ch08/code/s02-risk-ratio-calculator/README.md`

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

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

补充说明：

风险率计算要先确定 debt asset。借 quote 开多时，base 资产需要按价格换算成 quote；借 base 开空时，quote 资产需要换算成 base。价格方向搞反会直接导致健康度展示错误。

风险率不是只看 `BalanceManager` 可用余额，还要考虑 DeepBook Pool 中已结算但未提取的金额、订单相关余额以及随时间增长的 debt amount。清算机器人应使用链上读接口或 dev inspect，避免 indexer 延迟造成误判。

## 开发要点

- 显示风险率时同时展示 debt asset、债务金额、阈值和价格时间戳。
- 借款、提款、交易、TPSL 创建前都要模拟操作后的风险率。
- 把风险率小于 1、低于清算阈值、低于提款阈值分别映射成不同 UI 状态。

## 检查问题

- 当前风险率是用 base 计价还是 quote 计价？
- 哪些资产余额被计入 `assets_in_debt_unit`，open orders 是否已处理？
- 价格过期或 confidence 过大时，应用是阻止交易还是只给提示？
