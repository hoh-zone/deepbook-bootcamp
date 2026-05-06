# ch08-02 MarginRegistry

[返回本章](README.md)

## 本节目标

- 读懂 `MarginRegistry` 如何把版本、PoolConfig、MarginPool ID、manager 列表和维护者权限集中到一个控制平面。
- 能解释 `PoolConfig` 中四个风险率阈值和两个清算奖励如何影响借款、提款与清算。
- 能从 `marginPrep.ts` 还原部署时注册 DeepBook Pool、创建 MarginPool、启用借贷池的顺序。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/margin_registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_registry.move)
- [packages/deepbook_margin/sources/helper/margin_constants.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/margin_constants.move)
- [scripts/transactions/marginPrep.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/marginPrep.ts)

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

`MarginRegistry` 是所有 Margin 交易的控制平面。结构体在 `margin_registry.move` 中定义：

- `allowed_versions`：允许调用的 Margin package 版本。`init` 默认写入 `margin_constants::margin_version()`。
- `pool_registry`：DeepBook Pool ID 到 `PoolConfig` 的映射。
- `margin_pools`：资产类型 `TypeName` 到对应 `MarginPool<Asset>` ID 的映射。
- `margin_managers`：owner 地址到其 `MarginManager` ID 集合的映射。
- `allowed_maintainers` 和 `allowed_pause_caps`：维护者和暂停能力白名单。

`PoolConfig` 是每个 DeepBook Pool 的 Margin 风控配置，包含：

- `base_margin_pool_id`、`quote_margin_pool_id`：这个交易对可使用的 base/quote 借贷池。
- `RiskRatios`：`min_withdraw_risk_ratio`、`min_borrow_risk_ratio`、`liquidation_risk_ratio`、`target_liquidation_risk_ratio`。
- `user_liquidation_reward`、`pool_liquidation_reward`：清算者和借贷池奖励，使用 9 位小数定点数。
- `enabled`：是否允许常规 Margin 交易。

应用授权分两层：DeepBook 核心需要授权 Margin app，Margin 侧还要注册并启用具体 DeepBook Pool。部署脚本 [scripts/transactions/marginPrep.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/marginPrep.ts) 展示了顺序：`authorizeMarginApp`、创建 Pyth 配置、创建 MarginPool、`registerDeepbookPool`、`enableDeepbookPool`、`enableDeepbookPoolForLoan`。

补充说明：

`MarginRegistry` 是应用读取风控参数的首要来源。前端展示池状态时，不应把 DeepBook Pool ID、base/quote MarginPool ID 和风险率配置写死；这些值可能通过维护者交易被暂停、更新或重新授权。

借款和提款失败经常不是余额不足，而是 registry 中的池没有启用、版本不在 allowed list、DeepBook Pool 没有被对应 MarginPool 授权。交易前模拟要把这些配置状态作为独立检查项输出。

## 开发要点

- 把 PoolConfig 当作链上真值，SDK 配置只保存对象 ID 和类型，不缓存风险阈值作为执行依据。
- 部署或测试环境初始化时按 `marginPrep.ts` 的顺序执行，先授权 app，再创建池和注册交易对。
- 错误提示要区分 `pool_enabled` 失败、loan 未启用、版本不允许和风险率不足。

## 检查问题

- 当前 DeepBook Pool 的 base/quote MarginPool ID 来自 registry 还是本地配置？
- `min_borrow_risk_ratio` 和 `liquidation_risk_ratio` 分别在哪些操作中被检查？
- 如果 Pool 被暂停，用户还能通过哪些 reduce-only 路径降低风险？
