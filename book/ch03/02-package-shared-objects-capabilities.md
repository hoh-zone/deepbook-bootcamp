# ch03-02 package、shared object 与 capability 布局

[返回本章](README.md)

## 先抓住结构

先把“package、shared object 与 capability 布局”放进 DeepBook 的对象图里。这里不是罗列模块，而是建立阅读顺序：入口在哪里，状态放在哪里，资金最终在哪里结算。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/Move.toml)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 读架构

`registry.move` 的 `init` 创建并共享 `Registry`，同时把 `DeepbookAdminCap` 转给发布者。`pool.move` 的 `create_pool` 创建 `Pool<BaseAsset, QuoteAsset>` 后调用 `transfer::share_object(pool)`，所以每个交易对是共享对象。`balance_manager.move` 的 `BalanceManager` 也是 `has key, store` 的对象，可作为共享交易账户使用。

权限分成三层：

- 协议管理权限：`DeepbookAdminCap` 用于版本、白名单、注册表管理和 admin pool 创建。
- 应用授权：`Registry::authorize_app<App>` 写入 `AppKey<App>` 动态字段，允许应用访问受保护能力，例如 `new_with_custom_owner_caps`。
- 用户交易授权：`TradeCap`、`DepositCap`、`WithdrawCap` 绑定一个 `BalanceManager`，`TradeProof` 是可丢弃的临时证明，传入 `Pool` 后由 `Vault` 和 `BalanceManager` 校验。

交易前检查应包括：Pool 对象版本是否启用、BalanceManager 是否归属或有对应 cap、交易资产类型是否与池泛型一致、订单参数是否满足 tick/lot/min size。

## 阅读补充

DeepBook 的布局可以分三层看：package 提供不可变代码入口，shared object 承载 Pool/Registry 这类全局可访问状态，capability/proof 承载管理员、应用或用户交易授权。交易成功必须同时满足代码入口正确、共享对象版本可用、权限对象有效。

权限阅读建议从 struct 定义开始，再找创建函数、转移函数和校验函数。只有看到 cap 如何产生和在哪里被验证，才能判断它是否只是展示字段，还是实际保护了状态修改。

## 工程判断

- 交易构造文档要分开列 package ID、shared object ID 和 cap/proof。
- cap 的创建、转移、销毁和验证函数要成组阅读。
- 版本 abort 不属于余额问题，应在权限和对象检查前单独确认。

## 读完以后问自己

- package、shared object、capability 分别解决什么问题？
- `TradeProof` 在调用链中在哪里被验证？
- 为什么 package ID 正确仍可能因为 shared object 版本失败？
