# ch08-05 pool_proxy 如何连接 Margin 和 DeepBook Pool

[返回本章](README.md)

## 本节目标

- 读懂 `pool_proxy` 如何把 Margin 风控接到 DeepBook Pool 的现货撮合接口。
- 能说明限价单、市价单、撤单、结算和 reduce-only 入口分别检查什么。
- 能在应用设计中避免直接用 `BalanceManager` 绕过 Margin 价格保护。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/helper/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/oracle.move)

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

`pool_proxy.move` 是 Margin 应用交易时应该调用的交易入口，而不是直接拿 `BalanceManager` 调 DeepBook Pool。

常规下单路径：

1. 检查 `margin_manager.deepbook_pool() == pool.id()`。
2. 检查 `registry.pool_enabled(pool)`。
3. 对限价单调用 `registry.assert_price(pool.id(), price, is_bid, clock)`；对市价单先用订单簿计算 effective price，再做价格保护。
4. 从 `MarginManager` 取 `trade_proof` 和可变 `balance_manager`。
5. 调用 DeepBook Pool 的 `place_limit_order` 或 `place_market_order`。

当某个 Pool 被禁用 Margin 交易时，用户仍可能需要减少风险。`place_reduce_only_limit_order` 和 `place_reduce_only_market_order` 会读取当前债务和资产，只允许减少净负债方向的订单，否则抛出 `ENotReduceOnlyOrder`。

补充说明：

`pool_proxy` 的价值不只是代调用 DeepBook。它把 registry 的启用状态、oracle 价格保护、manager 的 `TradeCap` 和 reduce-only 判断放在同一条链上路径里，保证下单不会绕过 Margin 风控。

市价单需要先根据订单簿推导 effective price，再与 oracle 检查；限价单则直接检查用户给出的 price。应用侧因此要把滑点、订单簿深度和 oracle stale 风险都显示在交易预览中。

## 开发要点

- SDK 的交易模块只暴露 Margin 下单入口，不给普通用户暴露原始 DeepBook Pool 下单入口。
- Pool 暂停时把常规下单按钮切换为 reduce-only 平仓/降风险操作。
- 交易失败要区分价格保护失败、订单簿流动性不足、Pool 未启用和 manager-pool 不匹配。

## 检查问题

- 这笔订单会增加债务风险，还是符合 reduce-only 的降风险方向？
- 市价单的 effective price 是否经过 `registry.assert_price` 检查？
- 应用是否在下单后调用 withdraw settled amounts 或重新读取结算余额？
