# ch03 DeepBookV3 总体架构

## 本章目标

- 建立 DeepBookV3 的全局对象模型：`Pool`、`Book`、`State`、`Vault`、`BalanceManager`、`Registry` 分别承担什么职责。
- 能从一次下单交易反推源码调用链、资金结算链和事件链。
- 理解哪些状态集中在共享 `Pool` 上，哪些状态被抽象到 `BalanceManager`，以及这种拆分如何影响并发交易设计。
- 能解释 DeepBookV3 与 DeepBook Margin、DeepBook Predict 的依赖边界，而不是把不同协议的借贷和预测市场状态混在一起。

## 本章学习阶梯

- L2 先从一次下单的对象清单理解架构。
- L3 追 `Pool -> Book -> State -> Vault -> BalanceManager` 的主调用链。
- L4 解释版本、注册、capability 和对象借用如何影响真实交易。
- L5 能画出协议升级或交易失败时的排查路径。

## 源码地图

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：DeepBookV3 对外交易入口，定义 `Pool<BaseAsset, QuoteAsset>`、`PoolInner`、下单、交换、取消、治理、闪电贷入口。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：订单簿核心，定义 `Book`、双边订单容器、订单 ID 分配、撮合和挂单插入。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：挂在订单簿中的压缩订单状态，负责 maker fill、取消退款和订单事件。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：一次新订单的生命周期对象，记录 taker 执行状态、fills、费用、最终事件。
- [packages/deepbook/sources/book/fill.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/fill.move)：一次 taker 与 maker 订单匹配的成交结果。
- [packages/deepbook/sources/state/state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/state.move)：池内账户、历史、治理和费用状态处理。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：单个 `BalanceManager` 在池中的开放订单、已结算余额、交易量和 stake。
- [packages/deepbook/sources/state/governance.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/governance.move)：maker/taker fee、stake、投票和参数更新。
- [packages/deepbook/sources/state/history.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/history.move)：按 epoch 保存历史费率、交易量、rebate 等统计。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：池内资产保管、BalanceManager 结算和 DeepBookV3 闪电贷底层实现。
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：用户交易账户抽象，保存多资产余额并用 `TradeProof` 授权交易。
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)：池注册表、版本开关、稳定币白名单和应用授权。
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)：订单分页查询接口，服务前端和 SDK 查询。

## 小节目录

- [01 全景图：Pool、Book、State、Vault、BalanceManager、Registry](01-architecture-overview.md)
- [02 package、shared object 与 capability 布局](02-package-shared-objects-capabilities.md)
- [03 Pool 的泛型资产设计](03-pool-generic-assets.md)
- [04 Pool 是入口，Book 是撮合核心](04-pool-entry-book-matching-engine.md)
- [05 State：账户、历史、治理、费用、参数](05-state-accounts-history-governance-fees.md)
- [06 Vault：最终资产保管和结算](06-vault-custody-settlement.md)
- [07 BalanceManager：交易账户抽象](07-balance-manager-account-abstraction.md)
- [08 Registry：池注册、版本和权限](08-registry-pool-version-permissions.md)
- [09 DEEP token 在费用、staking、rebate 中的位置](09-deep-token-fees-staking-rebates.md)
- [10 对象借用顺序和并行边界](10-object-borrow-order-parallelism.md)
- [11 DeepBookV2 到 V3 的关键变化](11-v2-to-v3-changes.md)
- [12 DeepBookV3 与 Margin、Predict 的依赖关系](12-margin-predict-dependencies.md)
- [13 源码阅读路线](13-source-reading-route.md)
- [14 下单调用链练习](14-order-call-chain-exercise.md)

## 本章代码

- `code/s01-architecture-map/`：从源码提取模块依赖关系，生成架构图输入。
- `code/s02-pool-object-query/`：查询链上 Pool 对象字段，核对 `PoolInner` 中的 book/state/vault 结构。
- `code/s03-event-flow/`：按交易 digest 解析 DeepBook 事件，复原一次下单的事件链。

## Move 高阶穿插点

- 读架构时先画对象借用图：哪些是 `&mut` shared object，哪些只是只读参数，哪些资源会被消费。
- `Versioned`、`Registry` 和 capability 不是外围细节，它们决定生产环境升级、暂停和权限边界。
- 把每个模块都绑定到一种 Move 状态迁移：创建、借用、写入、拆分、合并、销毁或事件发出。

## 常见错误

- 错误：把 Pool 当成只包含订单簿的对象。实际 Pool 还包含 State、Vault、DeepPrice 和版本控制。
- 错误：下单前只检查钱包 coin。实际订单资金来自 BalanceManager，必须检查 BalanceManager 余额和 TradeProof 权限。
- 错误：认为 Book 负责费用和余额。Book 只负责撮合和挂单；费用在 State/OrderInfo 计算，资产在 Vault 结算。
- 错误：把 DeepBookV3 闪电贷与 Margin 借贷混写。V3 闪电贷入口在 `pool.move`，底层是 `vault.move` 的 hot potato。
- 错误：忽略版本开关。`Pool::load_inner` 和 `Registry::load_inner` 都可能因 package version 禁用而 abort。

## 本章检查清单

- [ ] 能画出 Pool、Book、State、Vault、BalanceManager、Registry 的关系。
- [ ] 能说出 `place_limit_order` 到 `vault::settle_balance_manager` 的完整调用链。
- [ ] 能解释 `Pool<BaseAsset, QuoteAsset>` 中 base/quote 泛型的意义。
- [ ] 能说明为什么 BalanceManager 是交易账户而不是钱包余额。
- [ ] 能区分 DeepBookV3 闪电贷和 DeepBook Margin 借贷。

## 进阶练习

- 练习 1：打开 `pool.move`，列出每个 public mutative 函数需要 mutable borrow 的对象，并判断它们的并发边界。
- 练习 2：从 `Registry::register_pool` 出发，解释为什么反向交易对不能重复注册。
- 练习 3：用一次部分成交订单，画出 `OrderInfo`、`Fill`、`State`、`Vault` 的状态变化表。

