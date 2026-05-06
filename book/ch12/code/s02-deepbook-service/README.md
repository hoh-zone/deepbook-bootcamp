# s02-deepbook-service

本示例封装 Spot 交易服务。它面向应用层暴露 `poolKey`、价格、数量和方向，不让业务代码直接处理 Move type、共享对象顺序或 SDK 子模块细节。

## 源码对应

- SDK 示例：[scripts/transactions/deepbookMarketMaker.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/deepbookMarketMaker.ts)
- BalanceManager 示例：[scripts/transactions/prepBalanceManager.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/prepBalanceManager.ts)
- Spot 入口：[packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- 资金账户：[packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)

## 服务接口

```ts
import type { Transaction } from "@mysten/sui/transactions";

export type LimitOrderInput = {
  poolKey: string;
  balanceManagerKey: string;
  clientOrderId: string;
  price: number;
  quantity: number;
  isBid: boolean;
  payWithDeep?: boolean;
};

export type MarketOrderInput = {
  poolKey: string;
  balanceManagerKey: string;
  quantity: number;
  isBid: boolean;
  payWithDeep?: boolean;
};

export type SwapInput = {
  poolKey: string;
  amount: number;
  minOut: number;
  direction: "baseToQuote" | "quoteToBase";
  deepAmount?: number;
};

export interface DeepBookService {
  createBalanceManager(): Transaction;
  deposit(balanceManagerKey: string, coinType: string, coinId: string): Transaction;
  withdraw(balanceManagerKey: string, coinType: string, amount: bigint): Transaction;
  placeLimitOrder(input: LimitOrderInput): Transaction;
  placeMarketOrder(input: MarketOrderInput): Transaction;
  cancelOrder(poolKey: string, balanceManagerKey: string, orderId: string): Transaction;
  cancelAll(poolKey: string, balanceManagerKey: string): Transaction;
  swap(input: SwapInput): Transaction;
}
```

## 初始化

```ts
import { Transaction } from "@mysten/sui/transactions";
import { createGrpcDeepBookClient, type DeepBookRuntimeConfig } from "../s01-sdk-init/config";

export class SdkDeepBookService implements DeepBookService {
  constructor(private readonly config: DeepBookRuntimeConfig) {}

  private sdk() {
    return createGrpcDeepBookClient(this.config);
  }

  private tx() {
    const tx = new Transaction();
    tx.setSender(this.config.address);
    return tx;
  }
}
```

## BalanceManager

`BalanceManager` 创建调用参考 `prepBalanceManager.ts`：

```ts
createBalanceManager() {
  const client = this.sdk();
  const tx = this.tx();
  client.deepbook.balanceManager.createAndShareBalanceManager()(tx);
  return tx;
}
```

deposit/withdraw 的 SDK 方法名会随版本变化，服务层可以先把能力收口到自己的接口。若 SDK 没有覆盖某个入口，就用 `tx.moveCall` 绑定 `balance_manager.move` 的 `deposit`、`withdraw`。

## 限价单

`pool.move` 的 `place_limit_order` 会写入订单簿和账户状态；SDK 封装示例见 `deepbookMarketMaker.ts`：

```ts
placeLimitOrder(input: LimitOrderInput) {
  const client = this.sdk();
  const tx = this.tx();

  tx.add(
    client.deepbook.deepBook.placeLimitOrder({
      poolKey: input.poolKey,
      balanceManagerKey: input.balanceManagerKey,
      clientOrderId: input.clientOrderId,
      price: input.price,
      quantity: input.quantity,
      isBid: input.isBid,
      payWithDeep: input.payWithDeep ?? true,
    }),
  );

  return tx;
}
```

交易前检查：

- `poolKey` 必须存在于 SDK/配置支持的池子中。
- `balanceManagerKey` 必须映射到当前用户可用的 BalanceManager。
- `price` 满足 tick size，`quantity` 满足 lot size 和 min size。
- 买单检查 quote/deep 余额，卖单检查 base/deep 余额。

## 市价单

```ts
placeMarketOrder(input: MarketOrderInput) {
  const client = this.sdk();
  const tx = this.tx();

  tx.add(
    client.deepbook.deepBook.placeMarketOrder({
      poolKey: input.poolKey,
      balanceManagerKey: input.balanceManagerKey,
      quantity: input.quantity,
      isBid: input.isBid,
      payWithDeep: input.payWithDeep ?? true,
    }),
  );

  return tx;
}
```

服务层需要明确 `quantity` 的单位。若业务 API 表达的是“最多支付多少 quote”，建议使用 swap exact input 接口，而不是市价单接口。

## 撤单和查询

撤单前先查询订单归属，避免用户误撤无关订单：

```ts
async function assertOwnOrder(poolKey: string, balanceManagerKey: string, orderId: string) {
  // 生产实现：调用 SDK 查询或 dev inspect `pool::get_order`。
  // 失败时抛出 ORDER_NOT_FOUND 或 ORDER_OWNER_MISMATCH。
}
```

批量撤单要设置最大订单数，例如每笔 PTB 最多 50 个 order ID。

## swap

```ts
swap(input: SwapInput) {
  const client = this.sdk();
  const tx = this.tx();

  const call =
    input.direction === "baseToQuote"
      ? client.deepbook.deepBook.swapExactBaseForQuote({
          poolKey: input.poolKey,
          amount: input.amount,
          deepAmount: input.deepAmount ?? 0,
          minOut: input.minOut,
        })
      : client.deepbook.deepBook.swapExactQuoteForBase({
          poolKey: input.poolKey,
          amount: input.amount,
          deepAmount: input.deepAmount ?? 0,
          minOut: input.minOut,
        });

  const [baseOut, quoteOut, deepOut] = tx.add(call);
  tx.transferObjects([baseOut, quoteOut, deepOut], this.config.address);
  return tx;
}
```

dry run 后检查 `balanceChanges`，确认输出不低于 `minOut`。
