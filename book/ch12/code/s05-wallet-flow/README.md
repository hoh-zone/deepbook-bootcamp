# s05-wallet-flow

本示例说明前端钱包交易流程。原则是：后端或前端可以构造 `Transaction`，但用户交易必须由钱包签名；服务端不托管用户私钥。

## 流程

1. 前端收集业务输入：pool、方向、价格、数量、BalanceManager。
2. 前端或后端构造 PTB。
3. 构造方设置 `sender` 为钱包地址。
4. 提交前 dry run，返回摘要。
5. 钱包签名并提交。
6. 等待 digest 确认，解析 effects、events、object changes、balance changes。

## 前端构造交易

```ts
import { Transaction } from "@mysten/sui/transactions";

export function buildLimitOrderTx(sdk: DeepBookClientLike, sender: string) {
  const tx = new Transaction();
  tx.setSender(sender);

  tx.add(
    sdk.deepBook.placeLimitOrder({
      poolKey: "SUI_DBUSDC",
      balanceManagerKey: "USER_MANAGER",
      clientOrderId: String(Date.now()),
      price: 1,
      quantity: 1,
      isBid: true,
      payWithDeep: true,
    }),
  );

  return tx;
}
```

## 钱包签名提交

```ts
const result = await wallet.signAndExecuteTransaction({
  transaction: tx,
  options: {
    showEffects: true,
    showEvents: true,
    showObjectChanges: true,
    showBalanceChanges: true,
  },
});

if (result.effects?.status.status !== "success") {
  throw new Error(result.effects?.status.error ?? "transaction failed");
}
```

## 后端构造，前端签名

后端接口只返回 bytes 或交易 JSON，不返回签名结果：

```ts
POST /api/deepbook/limit-order/build
{
  "sender": "0x...",
  "poolKey": "SUI_DBUSDC",
  "balanceManagerKey": "USER_MANAGER",
  "price": 1,
  "quantity": 1,
  "isBid": true
}
```

响应：

```json
{
  "transactionBytes": "AA...",
  "dryRun": {
    "status": "success",
    "gasUsed": "123456",
    "expectedBaseDelta": "0",
    "expectedQuoteDelta": "-1000000"
  }
}
```

前端把 bytes 交给钱包签名。后端不接收 `SUI_PRIVATE_KEY`，不代用户签名。

## 确认交易

```ts
const confirmed = await suiClient.waitForTransaction({
  digest: result.digest,
  options: {
    showEffects: true,
    showEvents: true,
    showBalanceChanges: true,
    showObjectChanges: true,
  },
});
```

确认后更新 UI 时不要只看 digest 存在，必须看 `effects.status.status`。失败交易也会有 digest。

## 用户侧错误

前端展示用户可理解的错误：

- `INSUFFICIENT_BALANCE`：BalanceManager 或钱包 coin 不足。
- `PRICE_TICK_INVALID`：价格不满足 tick size。
- `QUANTITY_LOT_INVALID`：数量不满足 lot size/min size。
- `ORDER_NOT_FOUND`：撤单目标不存在或不属于当前账户。
- `OBJECT_VERSION_EXPIRED`：共享对象版本冲突，刷新后重试。
- `WALLET_REJECTED`：用户拒签。

同时把原始 digest、Move abort 和 dry run 信息写入日志。
