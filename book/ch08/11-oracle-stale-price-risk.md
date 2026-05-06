# ch08-11 oracle 与 stale price 风险

[返回本章](README.md)

## 本节目标

- 理解 Margin 为什么依赖 Pyth 价格、confidence、freshness 和 EWMA 偏差检查。
- 能说明 oracle 价格在借款、提款、下单价格保护、风险率和清算中的作用。
- 能设计应用侧 stale price 提示和交易前模拟策略。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/helper/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/oracle.move)
- [packages/deepbook_margin/sources/margin_registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_registry.move)
- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)
- [scripts/transactions/marginPrep.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/marginPrep.ts)

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

`helper/oracle.move` 从 Pyth `PriceInfoObject` 读取价格，并检查：

- price feed ID 是否匹配资产类型。
- price 是否有效。
- confidence 是否超过 `max_conf_bps`。
- EWMA 价格偏差是否超过 `max_ewma_difference_bps`。
- price age 是否超过配置的 max age。

`pool_proxy.update_current_price` 会计算安全价格并写入 `MarginRegistry`，下单时 `assert_price` 用这个价格做偏离保护。`margin_manager.risk_ratio_unsafe` 和 `manager_state` 可用于读取展示，但开发者不能把 unsafe 结果当成链上执行一定成功的依据。

补充说明：

oracle 不是只服务 UI 估值，它是链上执行条件。`registry.assert_price` 会在交易入口检查价格是否可接受，`risk_ratio` 也要用价格把 base/quote 资产折算到 debt unit。

stale price 风险会同时影响误放大仓位和误清算。应用可以展示本地估值，但提交交易前必须通过 dry run 或 dev inspect 使用当前 price object，让链上 freshness 与 confidence 检查给出最终结果。

## 开发要点

- 价格组件展示 publish time、confidence 状态和与订单价格的偏差。
- 当 oracle stale 时禁用新增借款和开仓，但可考虑保留还款、补抵押等降风险操作。
- 不要用 indexer 缓存价格代替 PTB 中传入的 Pyth price object。

## 检查问题

- 交易失败是订单价格偏离 oracle，还是 oracle 本身 stale/confidence 不合格？
- 风险率计算是否使用了和链上同方向的 base/quote 换算？
- oracle 不可用时哪些用户操作仍应允许？
