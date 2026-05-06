# ch08-07 position_manager

[返回本章](README.md)

## 先看风险边界

读“position_manager”时先问：这一步会让账户风险变大还是变小，谁有权继续执行，失败时应该归因到价格、债务、池配置还是对象权限。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_pool/position_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/position_manager.move)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- [packages/deepbook_margin/sources/margin_pool/margin_state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/margin_state.move)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 读风险控制面

`position_manager.move` 不管理交易仓位，它管理供应者在 MarginPool 中的存款份额。`Position` 只有两个核心字段：

- `shares`：用户供应 shares。
- `referral`：供应 referral ID。

`increase_user_supply` 会在首次供应时添加 entry，然后增加 shares，并返回新的总 shares 和上一次 referral。`decrease_user_supply` 减少 shares，`user_supply_shares` 是读接口。

这个模块的设计重点是把“供应者权益”从 vault 金额中抽离。利息发生后，用户 shares 不变，shares 对应的资产金额通过 `margin_state.supply_shares_to_amount` 动态计算。

## 工程旁白

这个模块容易被误读为“仓位管理器”。实际上交易仓位在 `MarginManager` 和 DeepBook orders 中体现；`position_manager` 只记录谁给借贷池供应了多少 shares，以及 referral 归属。

供应者收益来自 shares 对应资产金额的增长，所以 indexer 不应只保存最初 deposit amount。展示供应仓位时要用最新 pool state 把 shares 换回 amount，并扣除提款限速和 vault 流动性的影响。

## 风控判断

- 供应仓位页命名为 lending position 或 supply position，避免与 margin trading position 混淆。
- 用 `SupplierCap` 作为供应者操作凭证，不能用 manager owner 代替。
- referral 变更要按源码返回的 previous referral 处理事件和统计。

## 动手检查

- 这个 position 是借贷池供应仓位还是杠杆交易仓位？
- 用户 shares 不变时，为什么可提 amount 会随利息变化？
- 提款失败时是 shares 不足、rate limit 还是 vault liquidity 不足？
