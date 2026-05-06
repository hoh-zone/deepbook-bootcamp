# ch08-01 Margin 与 Spot 的关系

[返回本章](README.md)

## 先看风险边界

读“Margin 与 Spot 的关系”时先问：这一步会让账户风险变大还是变小，谁有权继续执行，失败时应该归因到价格、债务、池配置还是对象权限。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- [packages/margin_liquidation/sources/liquidation_vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/margin_liquidation/sources/liquidation_vault.move)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 读风险控制面

DeepBook Margin 不重写订单簿。现货撮合仍由 DeepBook Pool 完成，Margin 只是在用户和 Pool 之间加了一层账户、借贷和风险控制。

资金路径分四段：

- 抵押：用户把 base、quote 或 DEEP 存入 `MarginManager`，内部调用 `BalanceManager.deposit_with_cap`。入口是 `margin_manager.move` 的 `deposit<BaseAsset, QuoteAsset, DepositAsset>`。
- 借贷：用户从 `MarginPool<BaseAsset>` 或 `MarginPool<QuoteAsset>` 借入资产，借来的 coin 立即存入同一个 `MarginManager`。入口是 `borrow_base` 和 `borrow_quote`。
- 交易：`pool_proxy.move` 使用 `MarginManager` 持有的 `TradeCap` 调用 DeepBook Pool 的 `place_limit_order`、`place_market_order`、`cancel_order` 和 `withdraw_settled_amounts`。
- 清算：当仓位风险率低于清算阈值时，清算方拿债务资产调用 `margin_manager::liquidate`，协议取消挂单、偿还部分或全部债务，并把用户资产按奖励规则转给清算方和借贷池。

这不是 DeepBookV3 闪电贷。DeepBook 闪电贷是 Pool 内的原子借还；Margin 借贷是 `MarginPool<Asset>` 的持续债务，债务以 borrow shares 记录，会随时间计息。

## 工程旁白

Margin 的核心边界是“债务生命周期”。DeepBook Pool 只看到一个带 `TradeCap` 的 BalanceManager 在下单；借款是否允许、借款后风险率是否足够、债务如何计息，全部在 Margin 模块内完成。应用侧如果绕过 `pool_proxy` 直接调用现货 Pool，就失去了价格保护、reduce-only 和 manager 权限约束。

和 DeepBookV3 闪电贷相比，Margin 借贷不会要求在同一笔 PTB 内归还。用户借入后会得到 borrow shares，后续利息由 `margin_state` 把 shares 换算成实时 debt amount；因此 UI、Indexer 和清算机器人都必须展示“债务资产”和“随时间增长的债务金额”。

## 风控判断

- 不要把 `MarginPool.supply` 作为用户保证金入口；保证金进入 `MarginManager.deposit`，供应资产进入借贷池 vault。
- 所有 Margin 下单都经过 `pool_proxy`，并在交易前重新读取 registry、oracle 和 pool 状态。
- 产品文案要把“杠杆借贷”与“闪电贷”分开，避免用户以为仓位会在交易结束时自动归还。

## 动手检查

- 这笔交易是否创建了持续债务，还是只是 DeepBook Pool 内的原子闪电贷？
- 下单路径是否经过 `pool_proxy` 并携带正确的 manager、registry、pool 和 oracle 对象？
- 风险率降低时，用户应该补抵押、还款、reduce-only 交易还是等待清算？
