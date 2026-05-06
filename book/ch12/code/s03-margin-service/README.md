# s03-margin-service

本示例封装 Margin 服务。它分成管理员/维护者接口和用户接口：管理员负责版本、价格配置、资金池；用户负责 MarginManager、抵押、借还、条件单和风险查询。

## 源码对应

- 管理脚本：[scripts/transactions/marginPrep.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/marginPrep.ts)
- 版本脚本：[scripts/transactions/enableMarginVersion.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/enableMarginVersion.ts)
- 供给脚本：[scripts/transactions/supplyToMarginPool.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/supplyToMarginPool.ts)
- 用户入口：[packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- 资金池入口：[packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)

## 配置

```ts
export type MarginConfig = {
  network: "mainnet" | "testnet";
  rpcUrl: string;
  address: string;
  marginAdminCap?: string;
  marginMaintainerCap?: string;
  supplierCapByAsset: Record<string, string>;
  marginPoolByAsset: Record<string, string>;
  pythStateId?: string;
};
```

配置值参考 `scripts/config/constants.ts` 的 `marginAdminCapID`、`marginMaintainerCapID`、`supplierCapID` 和各类 `*MarginPoolCapID`。如果测试网为空，初始化必须抛出 `CONFIG_MISSING`。

## 管理员接口

```ts
export class MarginAdminService {
  constructor(private readonly config: MarginConfig) {}

  enableVersion(version: number) {
    const client = createGrpcDeepBookClient(this.config);
    const tx = new Transaction();
    tx.setSender(this.config.address);
    client.deepbook.marginAdmin.enableVersion(version)(tx);
    return tx;
  }

  replacePythConfig() {
    const client = createGrpcDeepBookClient(this.config);
    const tx = new Transaction();
    tx.setSender(this.config.address);

    const pythConfig = client.deepbook.marginAdmin.newPythConfig(
      [
        { coinKey: "SUI", maxConfBps: 300, maxEwmaDifferenceBps: 1500 },
        { coinKey: "USDC", maxConfBps: 100, maxEwmaDifferenceBps: 500 },
      ],
      70,
    )(tx);

    client.deepbook.marginAdmin.removeConfig()(tx);
    client.deepbook.marginAdmin.addConfig(pythConfig)(tx);
    return tx;
  }
}
```

管理员交易建议复用 [scripts/utils/utils.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/utils/utils.ts) 的思路：设置 sender、gas payment、expiration，dry run 成功后再导出 bytes 给 multisig。

## 维护者接口

```ts
export function buildCreateMarginPool(config: MarginConfig, asset: "USDC" | "SUI") {
  const client = createGrpcDeepBookClient(config);
  const tx = new Transaction();
  tx.setSender(config.address);

  const protocolConfig = client.deepbook.marginMaintainer.newProtocolConfig(
    asset,
    {
      supplyCap: 2_000_000,
      maxUtilizationRate: 0.9,
      referralSpread: 0.2,
      minBorrow: 0.1,
      rateLimitCapacity: 400_000,
      rateLimitRefillRatePerMs: 0.018518,
      rateLimitEnabled: true,
    },
    {
      baseRate: 0,
      baseSlope: 0.15,
      optimalUtilization: 0.8,
      excessSlope: 5,
    },
  )(tx);

  client.deepbook.marginMaintainer.createMarginPool(asset, protocolConfig)(tx);
  return tx;
}
```

参数要跟风险团队确认后固化到配置，不要在 API 请求里允许任意用户传入。

## 供给资金池

`supplyToMarginPool.ts` 的核心调用是：

```ts
client.deepbook.marginPool.supplyToMarginPool(
  "USDC",
  tx.object(config.supplierCapByAsset.USDC),
  90_000,
)(tx);
```

服务层应在构造前检查 supplier cap 是否配置、asset 是否被允许、金额是否超过策略上限。

## 用户接口

用户侧绑定 `margin_manager.move` 的入口：

- `new` / `share`：创建并共享 MarginManager。
- `deposit`：向 manager 存入抵押资产。
- `withdraw`：提取可用资产。
- `borrow_base`、`borrow_quote`：从对应 margin pool 借资产。
- `repay_base`、`repay_quote`：偿还债务。
- `add_conditional_order`、`cancel_conditional_order`：条件单。
- `risk_ratio`、`can_place_limit_order`、`can_place_market_order`：交易前检查。

SDK 若没有覆盖某个用户入口，可以用 raw PTB：

```ts
tx.moveCall({
  target: `${config.marginPackageId}::margin_manager::deposit`,
  typeArguments: [baseType, quoteType, depositType],
  arguments: [
    tx.object(marginManagerId),
    tx.object(depositCoinId),
  ],
});
```

## 风控检查

每个用户写交易前必须检查：

- oracle 价格未过期，`max_price_age_ms` 和价格容忍度满足配置。
- 借款后风险率不超过协议阈值。
- margin pool 有可借流动性和 rate limit 容量。
- BalanceManager 中 DeepBook 挂单锁定余额不会导致提现或还款失败。
- 版本已启用，pool 被 Margin 允许交易。

dry run 失败时，把错误映射成 `ORACLE_STALE`、`INSUFFICIENT_COLLATERAL`、`BORROW_LIMIT`、`VERSION_DISABLED`、`POOL_NOT_ALLOWED` 等业务错误。
