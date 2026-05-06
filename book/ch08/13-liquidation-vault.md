# ch08-13 清算与 liquidation vault

[返回本章](README.md)

## 先看风险边界

读“清算与 liquidation vault”时先问：这一步会让账户风险变大还是变小，谁有权继续执行，失败时应该归因到价格、债务、池配置还是对象权限。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/margin_liquidation/sources/liquidation_vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/margin_liquidation/sources/liquidation_vault.move)
- [scripts/transactions/fundLiquidationVault.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/fundLiquidationVault.ts)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 读风险控制面

`margin_manager::liquidate` 是清算核心入口。流程是：

1. 校验 manager 属于对应 DeepBook Pool，债务来自传入的 MarginPool。
2. 计算风险率，要求 `registry.can_liquidate(pool.id(), risk_ratio)`。
3. 要求还款 coin 不低于 `min_liquidation_repay`。
4. 提取已结算金额并取消所有挂单。
5. 根据 debt、assets、用户奖励、池奖励和目标风险率计算 `repay_amount`。
6. 调用 `MarginPool.repay_liquidation` 降低 borrow shares。
7. 从 manager 中取出清算者应得资产，发出 `LiquidationEvent`。

`liquidation_vault.move` 是清算机器人常用的资金库。管理员用 `deposit<T>` 注入资产，用 `authorize_trader` 授权交易员。交易员调用 `liquidate_base` 或 `liquidate_quote`，vault 先取出债务资产，调用 `margin_manager.liquidate`，再把收到的 base、quote 和剩余 repay asset 存回 vault。

## 工程旁白

清算不是简单“拿全部抵押还全部债”。源码会根据当前风险率、目标风险率、债务资产、可用 collateral 和奖励参数计算应还金额；如果资产不足，可能出现 pool default 或坏账处理。

`liquidation_vault.move` 把清算资金集中到 vault，授权 trader 调用 `liquidate_base`/`liquidate_quote`，成功后把清算所得回收到 vault。机器人因此要同时扫描不健康 manager 和 vault 库存，而不是只看风险率。

## 风控判断

- 清算前用 dev inspect 估算 repay amount、奖励和清算后风险率。
- vault 需要按 debt asset 准备库存；base 债务和 quote 债务调用不同入口。
- 清算事件要记录 manager、pool、debt asset、repay amount、reward 和 default 情况。

## 动手检查

- 当前仓位低于 `liquidation_risk_ratio` 还是只是低于借款/提款阈值？
- 清算机器人是否有足够 debt asset，还是需要先 rebalance vault？
- 清算后目标是完全平仓还是回到 `target_liquidation_risk_ratio` 附近？
