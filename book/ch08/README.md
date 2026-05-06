# ch08 DeepBook Margin 协议源码精读

## 本章目标

官方文档把 DeepBook Margin 定位为 DeepBookV3 上的杠杆交易扩展：用户通过借入资金放大买卖能力，同时承担清算、利率和 oracle 风险。本章不从“多了哪些函数”开始，而是从风险系统开始读源码。

读完本章后，读者应该能解释 Margin 如何把抵押品、借贷池、DeepBook Spot 订单簿和清算流程连成一个杠杆交易系统；能读懂 `MarginRegistry`、`MarginManager`、`MarginPool<Asset>`、`pool_proxy`、`margin_state`、`protocol_config`、`tpsl` 和 `liquidation_vault` 的职责边界；也能根据风险率、利率、预言机价格和清算参数，推导一次开仓、还款或清算交易的状态变化。

## 官方风险基线

Margin 章节必须始终围绕风险写，而不是围绕接口列表写。

| 官方主题 | 本章落地方式 |
| --- | --- |
| Leveraged positions | ch09 讲用户路径，ch08 讲源码和状态边界。 |
| Risk management and liquidation | ch08-06、ch08-13、ch08-16 讲风险率、清算和阈值。 |
| Collateral flexibility | ch08-03、ch09-04 讲 `MarginManager` 如何包装抵押品和 BalanceManager。 |
| Interest accrual | ch08-08、ch08-09、ch08-10 讲 shares、utilization、protocol spread 和费用分配。 |
| Oracle risk | ch08-11 讲 stale price、confidence、EWMA 和交易前检查。 |

官方主网合同信息会随升级变化。本章解释对象和状态机时以源码为主；写对象 ID、版本、风险参数和池列表时，应回到 [DeepBook Margin contract information](https://docs.sui.io/onchain-finance/deepbook-margin/contract-information) 核对最新值。

## 本章学习阶梯

- L2 先理解 Margin 与 Spot 的关系：借贷和交易不是同一个状态机。
- L3 读 `MarginManager`、`MarginPool`、oracle、risk ratio 和 TPSL。
- L4 能解释借款、还款、清算和风险率变化的完整路径。
- L5 能为 Margin 协议做安全边界和异常路径审查。

## 关键定义卡片

Margin 的用户账户不是普通仓位对象，而是一个包装了 DeepBook 账户能力的共享对象：

```move
public struct MarginManager<phantom BaseAsset, phantom QuoteAsset> has key {
    id: UID,
    owner: address,
    deepbook_pool: ID,
    margin_pool_id: Option<ID>,
    balance_manager: BalanceManager,
    deposit_cap: DepositCap,
    withdraw_cap: WithdrawCap,
    trade_cap: TradeCap,
    borrowed_base_shares: u64,
    borrowed_quote_shares: u64,
    take_profit_stop_loss: TakeProfitStopLoss,
    extra_fields: VecMap<String, u64>,
}
```

这段定义解释了 Margin 为什么必须先懂 Spot：`MarginManager` 内部直接持有 `BalanceManager` 和 cap。Margin 交易通过这些能力调用 DeepBook 池，同时额外检查借贷 shares、oracle 和风险率。

借贷池定义：

```move
public struct MarginPool<phantom Asset> has key, store {
    id: UID,
    vault: Balance<Asset>,
    state: State,
    config: ProtocolConfig,
    protocol_fees: ProtocolFees,
    positions: PositionManager,
    allowed_deepbook_pools: VecSet<ID>,
    rate_limiter: RateLimiter,
    extra_fields: VecMap<String, u64>,
}
```

`vault` 保存供应资产，`state` 记录 shares 和利息，`allowed_deepbook_pools` 控制哪些 Spot 池可以接入这个借贷资产。Margin 的风险不是单个字段能解释的，必须把 manager、pool、registry 和 oracle 放在一起读。

## 源码地图

- [packages/deepbook_margin/sources/margin_registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_registry.move)：Margin 全局注册表，保存版本开关、DeepBook Pool 配置、MarginPool 映射、用户 manager 列表和价格保护数据。
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)：用户账户抽象，包装 DeepBook `BalanceManager`，记录 base/quote 借款 shares、TPSL 条件订单和仓位状态。
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)：单资产借贷池，处理供应、提取、借出、还款、协议费用、供应上限和提款限速。
- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)：Margin 到 DeepBook Pool 的交易代理，负责下单、撤单、结算、价格保护和 reduce-only 交易。
- [packages/deepbook_margin/sources/margin_pool/margin_state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/margin_state.move)：借贷池状态机，用 supply shares 和 borrow shares 累计利息。
- [packages/deepbook_margin/sources/margin_pool/protocol_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/protocol_config.move)：利率曲线、供应上限、最大利用率、协议 spread、最小借款和 rate limit 配置。
- [packages/deepbook_margin/sources/margin_pool/position_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/position_manager.move)：供应者持仓表，按 `SupplierCap` 记录 supply shares 和 referral。
- [packages/deepbook_margin/sources/margin_pool/protocol_fees.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/protocol_fees.move)：维护者费用、协议费用和供应 referral 费用分配。
- [packages/deepbook_margin/sources/helper/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/oracle.move)：Pyth 价格读取、置信度检查、EWMA 偏差检查和资产间价格换算。
- [packages/deepbook_margin/sources/tpsl.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/tpsl.move)：Take Profit / Stop Loss 条件订单的数据结构、排序、触发和事件。
- [packages/margin_liquidation/sources/liquidation_vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/margin_liquidation/sources/liquidation_vault.move)：清算资金库，给授权交易员提供还款资产，调用 `margin_manager::liquidate` 并回收清算所得。

## 小节目录

- [01 Margin 与 Spot 的关系](01-margin-vs-spot.md)
- [02 MarginRegistry](02-margin-registry.md)
- [03 MarginManager](03-margin-manager.md)
- [04 MarginPool&lt;Asset&gt;](04-margin-pool-asset.md)
- [05 pool_proxy 如何连接 Margin 和 DeepBook Pool](05-pool-proxy.md)
- [06 风险率](06-risk-ratio.md)
- [07 position_manager](07-position-manager.md)
- [08 margin_state 与利息累计](08-margin-state-interest.md)
- [09 protocol_config](09-protocol-config.md)
- [10 protocol_fees](10-protocol-fees.md)
- [11 oracle 与 stale price 风险](11-oracle-stale-price-risk.md)
- [12 TPSL 条件订单](12-tpsl-conditional-orders.md)
- [13 清算与 liquidation vault](13-liquidation-vault.md)
- [14 正常路径与失败边界](14-success-and-failure-paths.md)
- [15 开多和开空资金路径](15-long-short-fund-flows.md)
- [16 风险率跌破阈值后的处理流程](16-risk-ratio-below-threshold.md)

## 本章代码

- `code/s01-margin-source-map/`：生成 Margin 模块依赖图，帮助定位 registry、manager、pool、proxy、oracle、TPSL、liquidation vault 的调用关系。
- `code/s02-risk-ratio-calculator/`：用示例资产、债务和价格计算风险率，验证 min borrow、min withdraw 和 liquidation 阈值。
- `code/s03-interest-accrual/`：模拟 `margin_state.update` 和 `protocol_config.interest_rate`。
- `code/s04-liquidation-scenario/`：构造清算输入，推导 repay amount、用户奖励、池奖励和可能坏账。

## Move 高阶穿插点

- Margin 是复合状态机：抵押、借款、订单、风险率、oracle、清算每一层都有自己的不变量。
- 读 `MarginManager` 时先找它如何包装 `BalanceManager`，再看借贷 shares 和 TPSL 状态。
- oracle 相关代码要同时读价格值、decimals、时间戳和最大年龄，缺一个就不能做生产风控。

## 常见错误

- 把 `MarginPool.supply` 理解成给自己加保证金。供应到 MarginPool 是给借贷池提供流动性；用户保证金应存入 `MarginManager.deposit`。
- 直接调用 DeepBook Pool 下单。Margin 仓位必须通过 `pool_proxy`，否则价格保护、manager 权限和 reduce-only 逻辑不会生效。
- 用借款金额替代 borrow shares。链上债务随利息增长，真实金额要通过 `borrow_shares_to_amount` 计算。
- 只展示杠杆倍数，不展示风险率、利率、清算价和 oracle freshness。出版社级内容必须把用户能亏在哪里讲清楚。
- 忽略 oracle freshness。前端展示可以用 unsafe 读接口，但执行交易必须按链上安全 oracle 检查。
- 把清算奖励当成固定转账。`liquidate` 会根据目标风险率、资产覆盖程度和 repay coin 数量动态计算。

## 本章检查清单

- [ ] 能说明 `MarginRegistry` 中 PoolConfig 的每个风险参数如何影响借款、提款和清算。
- [ ] 能说明 `MarginManager` 为什么同时持有 `BalanceManager` 和三种 cap。
- [ ] 能说明 supply shares 与 borrow shares 为什么不会随每秒利息直接改变。
- [ ] 能追踪一次 `borrow_quote` 从 MarginPool vault 到 BalanceManager 的资金变化。
- [ ] 能解释 `pool_proxy` 对限价单和市价单做价格保护的差异。
- [ ] 能解释 TPSL 为什么设计成 permissionless execute。
- [ ] 能推导一次 partial liquidation 的 repay amount 和奖励。

## 进阶练习

- 画出 `MarginManager<SUI, USDC>` 开 2 倍 SUI 多头的对象图和资金流。
- 修改 `s02-risk-ratio-calculator` 的价格输入，观察风险率从 2.0 下降到 1.1 以下时哪些操作会失败。
- 用 `s03-interest-accrual` 比较利用率 40%、80%、95% 时的年化利率和一天后 borrow amount。
- 根据 `liquidation_vault.move` 设计一个清算机器人事件日志表，字段至少包含 manager、pool、repay asset、risk ratio、pool default。
