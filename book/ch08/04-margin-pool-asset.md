# ch08-04 MarginPool&lt;Asset&gt;

[返回本章](README.md)

## 本节目标

- 理解 `MarginPool<Asset>` 是单资产借贷池，负责供应、提款、借出、还款、利息和费用。
- 能区分供应者的 supply shares 与借款人的 borrow shares，并说明二者如何由 `margin_state` 换算。
- 能解释最大利用率、最小借款、供应上限和提款限速对应用体验的影响。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- [packages/deepbook_margin/sources/margin_pool/margin_state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/margin_state.move)
- [packages/deepbook_margin/sources/margin_pool/protocol_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/protocol_config.move)
- [packages/deepbook_margin/sources/margin_pool/position_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/position_manager.move)

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

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

补充说明：

`MarginPool` 同时服务两类用户：供应者把资产放入 vault 获取 supply shares，交易者通过 manager 借出资产形成 borrow shares。供应者赚取的不是固定利息转账，而是 shares 对应的可提金额随 `total_supply` 增加。

借款失败时要把池级限制和账户级限制分开：池级包括 vault 流动性、max utilization、min borrow 和 allowed DeepBook Pool；账户级包括借款后风险率。这样 UI 才能告诉用户是“池子没钱”还是“你的抵押不够”。

## 开发要点

- 供应和抵押是两个入口，供应资产不能直接提高某个 manager 的风险率。
- 提款前读取 rate limiter 和 vault 余额，避免把 shares 可换算金额当成一定可提金额。
- 清算还款走 `repay_liquidation`，它可能改变供应侧收益、奖励和坏账处理。

## 检查问题

- 用户看到的 APR 来自 `ProtocolConfig` 和当前 utilization，还是写死的显示值？
- withdraw 可提金额是否同时考虑 shares、vault balance 和 rate limit？
- 借款失败时应该检查 MarginPool 限制还是 manager 风险率，二者如何排序？
