# ch08-03 MarginManager

[返回本章](README.md)

## 本节目标

- 理解 `MarginManager<BaseAsset, QuoteAsset>` 是用户在单个 DeepBook Pool 上的 Margin 账户对象。
- 能说明 manager 为什么同时持有 DeepBook `BalanceManager`、三种 cap、债务 shares 和 TPSL 列表。
- 能判断创建、注册、注销、借款、还款、提款对 manager 状态的要求。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/margin_registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_registry.move)
- [packages/deepbook_margin/sources/tpsl.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/tpsl.move)

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 源码定义

`MarginManager` 的定义直接说明它为什么复杂：

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

字段分三层读：

- 账户层：`owner`、`balance_manager`、三种 cap。
- 市场层：`deepbook_pool`、`margin_pool_id`。
- 风险层：`borrowed_base_shares`、`borrowed_quote_shares`、`take_profit_stop_loss`。

这不是“一个仓位对象”，而是一个能代表用户调用 DeepBook、同时被 Margin 风控约束的共享账户。

## 正文

`MarginManager<BaseAsset, QuoteAsset>` 是一个共享对象，代表用户在某个 DeepBook Pool 上的 Margin 账户。它绑定：

- `owner`：只有 owner 可以存取、借款、还款、注册和注销 manager。
- `deepbook_pool`：这个 manager 只能服务一个 DeepBook Pool。
- `balance_manager`、`deposit_cap`、`withdraw_cap`、`trade_cap`：DeepBook 账户能力，由 manager 持有。
- `margin_pool_id`：当前债务来自哪个 MarginPool。源码限制一个 manager 同时只能从一个 MarginPool 借款。
- `borrowed_base_shares`、`borrowed_quote_shares`：债务份额，不直接存金额。
- `take_profit_stop_loss`：条件订单集合。

> **源码旁白**：`MarginManager` 是一个复合账户，不是单纯的“仓位对象”。它内部持有 DeepBook 的 `BalanceManager` 和三种 cap，所以 Margin 交易仍然沿用现货池的账户和权限模型，只是在外层叠加借贷和风险约束。

创建路径是 `new` 或 `new_with_initializer`。`new` 创建后直接 `share_object`，`new_with_initializer` 用 `ManagerInitializer` 强制调用方最后执行 `share`，避免 manager 被创建后没有共享。

注册路径是 `register_margin_manager`，它调用 `MarginRegistry.add_margin_manager`。注销路径是 `unregister_margin_manager`，源码要求 base/quote debt 都为 0，`margin_pool_id` 为空，base/quote/DEEP 余额也为 0。

补充说明：

`MarginManager` 的债务字段是 shares 而不是金额，这意味着“当前欠款”必须结合 `MarginPool` 的 `margin_state` 计算。应用侧展示 manager 时至少要同时读 manager、对应 MarginPool 和 oracle 价格，否则只能看到静态份额，无法得到真实风险率。

源码限制一个 manager 同时只从一个 MarginPool 借款，目的是让清算、还款和风险率计算维持明确的 debt asset。多市场应用如果希望用户交易多个池，应按 `(owner, pool)` 维护多个 manager，而不是把不同池仓位塞进同一个账户。

## 开发要点

- 创建 manager 后必须注册到 registry，方便应用和清算机器人枚举。
- 注销前检查 base debt、quote debt、margin_pool_id、base/quote/DEEP 余额和 open orders 都已清空。
- 不要在前端暴露 manager 持有的 cap；用户只签 PTB，cap 由共享对象内部使用。

## 检查问题

- 该 manager 绑定的是哪个 DeepBook Pool，是否和交易 PTB 传入的 Pool 一致？
- 当前 debt asset 是 base、quote 还是无债务，borrow shares 如何换算成 amount？
- 注销失败时，是债务未清、余额未提、挂单未结算还是 manager 未从 registry 移除？
