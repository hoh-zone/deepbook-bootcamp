# s04-predict-service

本示例给出 Predict 服务封装骨架。Predict 当前应以 Move 包实际发布状态为准，配置中必须显式写出网络、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。不要假设它已经拥有与 Spot 完全相同的稳定 SDK 表面。

## 源码对应

- Registry：[packages/predict/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/registry.move)
- Predict 主入口：[packages/predict/sources/predict.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/predict.move)
- PredictManager：[packages/predict/sources/predict_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/predict_manager.move)
- RangeKey：[packages/predict/sources/market_key/range_key.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/market_key/range_key.move)
- Oracle：[packages/predict/sources/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/oracle.move)

## 配置

```ts
export type PredictConfig = {
  network: "mainnet" | "testnet";
  rpcUrl: string;
  address: string;
  packageId: string;
  registryId: string;
  predictId: string;
  quoteCoinType: string;
  clockId: "0x6";
  oracleByAsset: Record<string, string>;
  versionLabel: string;
};
```

示例：

```ts
export const predictConfig: PredictConfig = {
  network: "testnet",
  rpcUrl: "https://fullnode.testnet.sui.io:443",
  address: "0x...",
  packageId: "0x...",
  registryId: "0x...",
  predictId: "0x...",
  quoteCoinType: "0x...::usdc::USDC",
  clockId: "0x6",
  oracleByAsset: { BTC: "0x..." },
  versionLabel: "predict package under active migration; verify before mainnet use",
};
```

## 服务接口

```ts
import { Transaction } from "@mysten/sui/transactions";

export class PredictService {
  constructor(private readonly config: PredictConfig) {}

  private tx(sender = this.config.address) {
    const tx = new Transaction();
    tx.setSender(sender);
    return tx;
  }
}
```

## PredictManager

`predict_manager.move` 中 `PredictManager` 保存用户资产和 range position。若当前发布包通过 registry 创建 manager，服务层应封装成一个方法，并隐藏 derived object 细节：

```ts
createPredictManager() {
  const tx = this.tx();
  tx.moveCall({
    target: `${this.config.packageId}::registry::create_predict_manager`,
    arguments: [tx.object(this.config.registryId)],
  });
  return tx;
}
```

具体 target 要以当前发布的 `registry.move` 入口为准；如果发布版本没有 public entry，应由后端在初始化阶段创建并记录 manager object ID。

## deposit 和 withdraw

`predict_manager.move` 提供 `deposit<T>` 和 `withdraw<T>`：

```ts
deposit(managerId: string, coinId: string) {
  const tx = this.tx();
  tx.moveCall({
    target: `${this.config.packageId}::predict_manager::deposit`,
    typeArguments: [this.config.quoteCoinType],
    arguments: [
      tx.object(managerId),
      tx.object(coinId),
    ],
  });
  return tx;
}

withdraw(managerId: string, amount: bigint) {
  const tx = this.tx();
  const coin = tx.moveCall({
    target: `${this.config.packageId}::predict_manager::withdraw`,
    typeArguments: [this.config.quoteCoinType],
    arguments: [
      tx.object(managerId),
      tx.pure.u64(amount),
    ],
  });
  tx.transferObjects([coin], this.config.address);
  return tx;
}
```

## mint、redeem 和 supply

`predict.move` 中的核心入口包括 `mint<Quote>`、`redeem<Quote>`、`redeem_permissionless<Quote>`、`supply<Quote>`、`withdraw<Quote>`。服务层应把 range 参数组合成明确的业务输入：

```ts
export type MintRangeInput = {
  managerId: string;
  oracleId: string;
  expiryMs: bigint;
  lowerStrike: bigint;
  higherStrike: bigint;
  quoteAmount: bigint;
};

mintRange(input: MintRangeInput) {
  const tx = this.tx();
  tx.moveCall({
    target: `${this.config.packageId}::predict::mint`,
    typeArguments: [this.config.quoteCoinType],
    arguments: [
      tx.object(this.config.predictId),
      tx.object(input.managerId),
      tx.object(input.oracleId),
      tx.pure.u64(input.expiryMs),
      tx.pure.u64(input.lowerStrike),
      tx.pure.u64(input.higherStrike),
      tx.pure.u64(input.quoteAmount),
      tx.object(this.config.clockId),
    ],
  });
  return tx;
}
```

上面的参数顺序是服务骨架，落地时必须和当前 `predict.move` 函数签名逐项核对。

## quote 和 oracle 检查

报价不要直接依赖前端本地计算。推荐顺序：

1. 读取 `Predict`、`OracleSVI`、vault 和 pricing config。
2. 用 dev inspect 调用只读报价入口，例如 `quote_unit_price`。
3. 检查 oracle 更新时间、ask bounds、trading paused、vault 可用余额。
4. 把报价快照和交易 PTB 一起返回给钱包签名。

错误映射建议：

- `PREDICT_NOT_CONFIGURED`：package/object ID 缺失。
- `ORACLE_STALE`：价格源过期。
- `TRADING_PAUSED`：交易暂停。
- `ASK_OUT_OF_BOUNDS`：报价超出配置范围。
- `VAULT_CAPACITY`：vault 风险额度不足。

所有错误都应保留原始 `effects.status.error`，方便追踪 Move abort code。
