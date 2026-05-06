# s03 Pricing calculator

目标：实现一个简化版 fee 计算器，对照 [packages/predict/sources/config/pricing_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/config/pricing_config.move)。

源码公式：

```text
variance = fair_price * (1 - fair_price)
bernoulli_factor = sqrt(variance)
bernoulli_fee = base_fee * bernoulli_factor
utilization_fee = base_fee * utilization_multiplier * min(liability / balance, 1)^2
fee_rate = max(bernoulli_fee, min_fee) + utilization_fee
```

注意所有价格和比例使用 `FLOAT_SCALING = 1_000_000_000`。`fee_rate` 是每单位绝对价格增量，不是 bps。

最小 TypeScript 骨架：

```ts
const SCALE = 1_000_000_000n;

function mul(a: bigint, b: bigint) {
  return (a * b) / SCALE;
}

function utilizationFee(baseFee: bigint, multiplier: bigint, liability: bigint, balance: bigint) {
  if (balance === 0n || liability === 0n) return 0n;
  const util = liability >= balance ? SCALE : (liability * SCALE) / balance;
  return mul(baseFee, mul(multiplier, mul(util, util)));
}
```

扩展练习：补上整数 sqrt，并用 `pricing_config_tests.move` 里的 half price、min fee、full utilization 案例做断言。

建议运行方式：

```bash
pnpm init
pnpm add -D typescript tsx vitest @types/node
pnpm exec vitest run
```
