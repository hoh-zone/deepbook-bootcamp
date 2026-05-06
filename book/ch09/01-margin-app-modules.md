# ch09-01 Margin 交易应用模块设计

[返回本章](README.md)

## 先看用户路径

这一节从 Margin 用户动作读起：围绕“Margin 交易应用模块设计”，先把 manager、collateral、loan、trade 和 repay 的顺序排清楚，再看源码入口。

## 源码入口

重点阅读：

- [scripts/transactions/marginPrep.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/marginPrep.ts)
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)
- [packages/margin_liquidation/sources/liquidation_vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/margin_liquidation/sources/liquidation_vault.move)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 把 PTB 串起来

一个可用的 Margin 应用至少拆成六个模块：

- 配置模块：网络、DeepBook package、Margin package、registry、Pyth price object、coin type、Pool key、MarginPool ID。
- 账户模块：查询、创建、注册、注销 `MarginManager`，维护 owner 到 manager ID 的映射。
- 资金模块：抵押品存入和提取，供应资产到 MarginPool，展示 vault liquidity 和用户 supply shares。
- 交易模块：借入 base/quote，调用 `pool_proxy` 下限价单、市价单、撤单和结算。
- 风控模块：风险率、健康度、利息、可借额度、可提额度、TPSL 状态和清算阈值。
- 机器人模块：TPSL 触发、清算扫描、liquidation vault 余额和授权 trader 管理。

后端不要把“估算成功”当成“交易一定成功”。每个写交易都要执行 dry run 或 dev inspect，并把 Move abort code 映射成用户能理解的错误。

## 工程旁白

应用架构的关键是把“配置真值”和“用户会话状态”分开。registry、PoolConfig、MarginPool 和 oracle object 是链上真值；本地 SDK 配置只是帮助构造 PTB；Indexer 缓存只用于查询加速。

交易模块不要暴露 DeepBook 原始下单作为 Margin 下单的快捷方式。Margin 持续债务必须通过 manager 和 `pool_proxy` 进入现货池；DeepBookV3 闪电贷如果用于其他场景，也应作为独立功能，不和 margin borrow/repay 共享状态。

## Margin 应用判断

- 每个写操作都生成 PTB、dry run、错误映射和事件刷新四个步骤。
- 账户模块按 `(owner, poolKey)` 组织 manager，避免多池仓位混淆。
- 机器人模块需要独立密钥和权限，不能复用用户签名或 admin cap。

## 动手检查

- 这个功能读的是链上真值、Indexer 派生状态还是前端缓存？
- 用户操作会产生持续债务吗，是否需要展示 interest accrual？
- 机器人执行 TPSL/清算时，失败事件如何回流到 UI？
