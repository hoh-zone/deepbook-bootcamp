# s02 Market key builder

目标：说明应用如何构造 Predict market key。源码入口是 [packages/predict/sources/market_key/range_key.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/market_key/range_key.move)。

二元 UP：

```ts
const lower = strike;
const higher = (1n << 64n) - 1n;
```

二元 DOWN：

```ts
const lower = 0n;
const higher = strike;
```

区间市场：

```ts
const lower = 90_000n * 1_000_000_000n;
const higher = 110_000n * 1_000_000_000n;
```

PTB 中需要调用：

```ts
tx.moveCall({
  target: `${PACKAGE_ID}::range_key::new`,
  arguments: [
    tx.pure.id(oracleId),
    tx.pure.u64(expiryMs),
    tx.pure.u64(lower),
    tx.pure.u64(higher),
  ],
});
```

生产检查：

- `lower < higher`。
- 不允许 `(-inf, +inf]`。
- strike 必须落在 oracle grid 内，并按 tick size 对齐。
- market key 必须绑定 oracle ID 和 expiry，不能只按 asset/strike/direction 缓存。
