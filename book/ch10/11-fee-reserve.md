# ch10-11 fee reserve 和协议收费路径

[返回本章](README.md)

## 先看市场问题

读“fee reserve 和协议收费路径”时先抓参数边界。Predict 的关键不是某个函数名，而是 oracle、expiry、strike、vault 和 payout 之间的关系。

## 源码入口

- `packages/predict/sources/accounting/fee_reserve.move`：fee reserve 结构和拆分。
- `packages/predict/sources/predict.move`：mint/redeem 时调用 fee reserve 与 vault 的路径。
- `packages/predict/sources/config/pricing_config.move`：fee rate 来源。

## 读市场参数

`fee_reserve.move` 把 charged trade fee 拆成 LP、protocol 和 insurance 三部分。默认比例来自 constants，测试显示 100 fee units 会按默认 60/20/20 拆分，101 fee units 的 rounding dust 留给 LP。`predict.move::apply_fee` 把 protocol 和 insurance 份额存入 reserve，把 LP 份额重新存入 vault，因此 LP fee 会留在 NAV 中。

settlement redeem 不收费，`apply_fee` 遇到 0 fee 会直接销毁 zero balance，不发 fee accrual。事件层面，`PositionMinted` 和 `PositionRedeemed` 都带 `fee_amount`，`FeeAccrued` 带 quote asset、total fee 和三路拆分。

mint 的 all-in cost 可以拆成 principal、LP fee、protocol fee、insurance fee。principal 和 LP fee 进入 vault 后影响 LP NAV；protocol/insurance fee 进入 `FeeReserve`，后续提取需要单独权限和事件记录。

财务报表不要只记录总 fee。至少要按 market key、oracle、expiry、quote asset、fee bucket 和交易 digest 归档。由于 Predict Indexer 仍是未来需求，当前可以在应用侧保存提交摘要，但不能声称已有稳定索引表。

## Predict 边界判断

- 报价响应展示 principal、LP fee、protocol fee、insurance fee 和 all-in ask。
- 报表口径明确 vault 收益与 protocol/insurance reserve 的边界。
- 错误解析把 fee 过高、ask bound 失败和余额不足拆开提示。

## 动手检查

- 哪些 fee 会提高 PLP NAV，哪些不会？
- 为什么 settled redeem 不应再收取交易费？
- 如果未来做 fee Indexer，主键应包含哪些 market 字段？
