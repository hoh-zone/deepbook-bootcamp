# ch03-08 Registry：池注册、版本和权限

[返回本章](README.md)

## 先抓住结构

先把“Registry：池注册、版本和权限”放进 DeepBook 的对象图里。这里不是罗列模块，而是建立阅读顺序：入口在哪里，状态放在哪里，资金最终在哪里结算。

## 架构坐标

先把架构坐标钉在这些文件上。读这一节时不需要展开所有实现，只要看清对象分工、入口函数和后续会反复回来的状态边界。

- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move)
- [scripts/transactions](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions)

这些入口是架构地图上的锚点。先标注每个模块负责的状态，再顺着一次下单或结算回到具体函数。

## 读架构

`Registry` 的 `RegistryInner` 保存 `allowed_versions`、`pools: Bag` 和 `treasury_address`。池注册使用 `PoolKey { base, quote }`，并且 `register_pool` 同时检查反向交易对是否已存在，避免同一对资产出现 `A/B` 和 `B/A` 两个独立池。

`Registry::load_inner` 和 `Pool::load_inner` 都检查当前 package version 是否在 allowed set 中。应用集成时，遇到版本禁用 abort 不应重试同一交易，而应刷新包配置、池 ID 和 SDK 常量。

## 阅读补充

Registry 负责回答“这个池是否被协议承认、当前版本是否可用、哪些应用或资产被授权”。它不撮合订单，但它的注册和版本检查会影响交易入口能否继续执行。

阅读 Registry 时重点看注册池、版本加载、稳定币/白名单和 `authorize_app` 相关动态字段。运维脚本通常会调用这些能力，但脚本只是入口，真正权限仍在 Move 代码里。

## 工程判断

- 新增池或切换池前先查 Registry，而不是只保存 Pool ID。
- 版本禁用应展示为协议维护/升级类错误，不要提示余额不足。
- 应用授权要查动态字段和 cap，不要只看前端配置。

## 读完以后问自己

- Registry 解决哪些 Pool 自身不适合解决的问题？
- 反向交易对重复注册为什么需要检查？
- 版本开关失败和交易参数失败如何区分？
