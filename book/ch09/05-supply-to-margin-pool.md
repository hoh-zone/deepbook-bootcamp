# ch09-05 供应资产到 MarginPool

[返回本章](README.md)

## 本节目标

- 实现供应者向 `MarginPool<Asset>` 存入资产并获得 supply shares 的流程。
- 能展示供应 APR、utilization、supply cap、vault liquidity 和可提金额。
- 能避免把 supply 操作误导为给交易账户加保证金。

## 源码关联

重点阅读：

- [scripts/transactions/supplyToMarginPool.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/supplyToMarginPool.ts)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- [packages/deepbook_margin/sources/margin_pool/position_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/position_manager.move)
- [packages/deepbook_margin/sources/margin_pool/margin_state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/margin_state.move)

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

供应资产是成为借贷池 LP，不是给自己的 manager 加保证金。参考 [scripts/transactions/supplyToMarginPool.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/supplyToMarginPool.ts)：

```ts
client.deepbook.marginPool.supplyToMarginPool(
  "USDC",
  tx.object(supplierCapID[env]),
  90_000,
)(tx);
```

链上 `MarginPool.supply` 会：

- 调 `state.increase_supply` 更新利息。
- 增加供应者 supply shares。
- 更新 referral fee shares。
- 把 coin 加入 vault。
- 记录 rate limiter deposit。
- 检查 `total_supply <= supply_cap`。

供应页面需要展示：

- 用户 supply amount 和 supply shares。
- 池 total supply、total borrow、utilization、interest rate。
- supply cap 剩余额度。
- withdrawal rate limit 和当前可提。
- referral 收益，如果应用支持 referral。

补充说明：

`supplyToMarginPool.ts` 对应的是借贷池资金端。用户供应后拿到的是供应 position 和 shares，收益来自借款人利息；它不创建 MarginManager 债务，也不会让某个杠杆仓位更安全。

供应页面的风险提示应包含 utilization 和提款限制。高 utilization 可能带来更高 borrow APR 和 supply APR，但也意味着 vault 可提流动性更紧张，rate limit 可能让大额提款需要等待。

## 开发要点

- 供应入口使用 `SupplierCap`/position 语义，不依赖交易 manager。
- 可提金额用 shares 实时换算，并额外显示 rate limit 和 vault balance。
- 供应成功后刷新 lending position，而不是 margin trading position。

## 检查问题

- 供应资产后，用户获得的是 supply shares 还是抵押余额？
- 当前 pool 是否接近 max utilization，提款是否可能受限？
- APR 展示是否来自 `ProtocolConfig.interest_rate` 和当前 utilization？
