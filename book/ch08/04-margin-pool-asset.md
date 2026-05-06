# ch08-04 MarginPool&lt;Asset&gt;

[返回本章](README.md)

## 先看风险边界

这里先把问题放到风险控制面里。“MarginPool&lt;Asset&gt;”不是在 DeepBook 旁边加一层 UI，而是把 registry、manager、oracle、borrow/supply state 接进同一条链上风控路径。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- [packages/deepbook_margin/sources/margin_pool/margin_state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/margin_state.move)
- [packages/deepbook_margin/sources/margin_pool/protocol_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/protocol_config.move)
- [packages/deepbook_margin/sources/margin_pool/position_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/position_manager.move)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 关键定义

`MarginPool` 的核心字段如下：

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

字段要按三组读：

- 资金和份额：`vault`、`state`、`positions`。
- 风险和限制：`config`、`allowed_deepbook_pools`、`rate_limiter`。
- 收费和扩展：`protocol_fees`、`extra_fields`。

供应入口是公开函数，借款入口是包内函数。这一点很重要：普通用户可以 supply，但借款必须通过 `MarginManager` 和协议内部风控路径进入，不能绕过 manager 直接从池子借资产。

## 读风险控制面

`MarginPool<Asset>` 是一个单资产借贷池。核心字段包括：

- `vault`：真实资产余额。
- `state`：`margin_state::State`，保存 total supply、total borrow 和 shares。
- `config`：`ProtocolConfig`，保存利率曲线、供应上限、最大利用率、协议 spread、最小借款和 rate limit。
- `positions`：供应者 shares 表。
- `protocol_fees`：维护者、协议和 referral 费用。
- `allowed_deepbook_pools`：哪些 DeepBook Pool 的 manager 可以向本池借款。
- `rate_limiter`：控制提款速率。

供应者需要 `SupplierCap`。`supply` 接收 `Coin<Asset>`、`SupplierCap` 和 referral，调用 `state.increase_supply` 获得 supply shares，再把 coin 加入 vault。`withdraw` 用用户 shares 换算可提金额，检查 rate limit 和 vault 余额后返回 coin。

借款入口是 `public(package) fun borrow`，只有 Margin package 内部能调用。它检查：

- 借款金额不低于 `min_borrow`。
- 借款后利用率不超过 `max_utilization_rate`。
- DeepBook Pool 已被允许向该 MarginPool 借款。
- vault 中有足够资产可 split。

还款分普通还款和清算还款：`repay` 按 shares 减少债务，`repay_liquidation` 还会把奖励、坏账或额外资产反映到 supply 侧。

## 工程旁白

`MarginPool` 同时服务两类用户：供应者把资产放入 vault 获取 supply shares，交易者通过 manager 借出资产形成 borrow shares。供应者赚取的不是固定利息转账，而是 shares 对应的可提金额随 `total_supply` 增加。

借款失败时要把池级限制和账户级限制分开：池级包括 vault 流动性、max utilization、min borrow 和 allowed DeepBook Pool；账户级包括借款后风险率。这样 UI 才能告诉用户是“池子没钱”还是“你的抵押不够”。

## 风控判断

- 供应和抵押是两个入口，供应资产不能直接提高某个 manager 的风险率。
- 提款前读取 rate limiter 和 vault 余额，避免把 shares 可换算金额当成一定可提金额。
- 清算还款走 `repay_liquidation`，它可能改变供应侧收益、奖励和坏账处理。

## 动手检查

- 用户看到的 APR 来自 `ProtocolConfig` 和当前 utilization，还是写死的显示值？
- withdraw 可提金额是否同时考虑 shares、vault balance 和 rate limit？
- 借款失败时应该检查 MarginPool 限制还是 manager 风险率，二者如何排序？
