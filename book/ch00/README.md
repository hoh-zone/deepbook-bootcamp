# ch00 DeepBook 是什么

## 本章目标

读完本章后，你应该能够：

- 用产品语言解释 DeepBook 是什么、解决什么问题，以及为什么它不是 AMM。
- 把 DeepBookV3、DeepBook Margin、DeepBook Predict、SDK、Indexer、Server 放进同一张功能地图。
- 说明现货订单簿、`BalanceManager`、费用、staking、治理、闪电贷、Margin 和 Predict 分别属于哪一层。
- 明确本书后续章节会深入哪些源码，哪些内容只作为边界说明。

## 本章学习阶梯

- L1 先回答“DeepBook 到底是什么”，不用读源码。
- L2 用功能地图区分 Spot、Margin、Predict、SDK、Indexer 和 Server。
- L3 找到每个功能背后的源码入口，但暂时只建立边界。
- L4 根据自己的目标选择后续阅读路线。

## 源码地图

本章只做功能总览，不做完整源码精读。所有链接都指向本书当前锁定的 DeepBook 源码提交。

| 功能域 | GitHub 源码 | 本章关注点 |
| --- | --- | --- |
| DeepBook 总说明 | [README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/README.md) | CLOB 定位、并行执行、低费用、闪电贷、治理、BalanceManager、DEEP |
| V3 现货核心 | [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) | 下单、撤单、swap、staking、治理、闪电贷、查询入口 |
| 账户抽象 | [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) | owner、trader、cap、余额托管和权限证明 |
| 资产保管和闪电贷 | [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) | `FlashLoan`、borrow/return、结算、DEEP burn |
| Margin | [packages/deepbook_margin/sources](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources) | `MarginManager`、借贷池、oracle、风险率、TPSL |
| Predict | [packages/predict/sources](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources) | 预测市场、区间头寸、LP vault、oracle、费用储备 |
| Indexer | [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer) | 链上事件落库、订单成交、Margin 事件、闪电贷事件 |
| Server | [crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server) | REST 查询、行情、订单簿、成交、Margin 和组合数据 |

## 小节目录

- [01 DeepBook 解决什么问题](01-what-is-deepbook.md)
- [02 为什么是链上中央限价订单簿](02-why-onchain-clob.md)
- [03 功能地图](03-feature-map.md)
- [04 现货交易能力](04-spot-trading-capabilities.md)
- [05 BalanceManager 账户模型](05-balance-manager-account-model.md)
- [06 Pool、Book、State、Vault 的分工](06-pool-book-state-vault.md)
- [07 DEEP、费用、staking 与治理](07-deep-fees-staking-governance.md)
- [08 闪电贷在源码中的位置](08-flashloans.md)
- [09 DeepBook Margin 是什么](09-margin-overview.md)
- [10 DeepBook Predict 是什么](10-predict-overview.md)
- [11 SDK、Indexer 与 Server](11-sdk-indexer-server.md)
- [12 可以构建哪些应用](12-application-scenarios.md)
- [13 本书阅读路线](13-reading-roadmap.md)

## 本章代码

- `code/s01-feature-map/`：把 DeepBook 功能域、源码路径、后续章节映射整理成一份可维护清单。

## Move 高阶穿插点

- 把 DeepBook 当作大型 Move 案例阅读：先判断对象所有权，再判断函数是否迁移状态。
- 看到产品能力时同步问一句：它最终依赖哪种 Move 资源约束，是 `key` 对象、泛型资产、cap，还是 hot potato。
- 读功能地图时不要只背模块名，要把每个模块关联到一个失败场景，例如权限错误、版本错误、资产方向错误或资源未消费。

## 常见错误

- 只把 DeepBook 理解成“swap 接口”。DeepBook 的底层是订单簿，swap 是构造市价吃单的一种应用入口。
- 把 `deepbook_margin` 的借贷能力误认为 V3 spot 的普通订单能力。Margin 是独立包，调用 DeepBook 池完成杠杆交易。
- 认为源码里没有闪电贷。当前锁定源码中，`pool.move` 暴露 `borrow_flashloan_base`、`borrow_flashloan_quote`、`return_flashloan_base`、`return_flashloan_quote`，`vault.move` 持有具体实现。
- 把 Indexer 当作链上真实状态。Indexer 是查询层，最终事实仍然来自链上对象、交易和事件。
- 在产品设计阶段忽略 `BalanceManager`。大多数非 swap 交互都围绕它组织资金和权限。

## 本章检查清单

- [ ] 能用三句话解释 DeepBook 的产品定位。
- [ ] 能列出 DeepBookV3 的现货核心能力。
- [ ] 能说明 Margin 和 Predict 与 V3 spot 的关系。
- [ ] 能指出闪电贷源码所在的两个关键文件。
- [ ] 能判断一个应用应该直接读链、接 SDK、接 Indexer，还是自建 Server。

## 进阶练习

1. 画一张 DeepBook 功能地图，把用户、做市商、协议、Indexer、Server、SDK 和前端放进去。
2. 阅读 [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)，只记录所有 `public fun` 的名字，并按“交易、治理、查询、管理”分类。
3. 阅读 [crates/indexer/src/handlers](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers)，找出哪些事件服务现货，哪些事件服务 Margin。
