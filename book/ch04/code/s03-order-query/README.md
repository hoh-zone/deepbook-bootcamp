# s03-order-query

目标：查询链上订单页，并验证本地 order id 解码和排序逻辑。

环境变量：

```bash
export SUI_RPC_URL=https://fullnode.mainnet.sui.io:443
export POOL_ID=0x...
```

建议依赖：

```bash
pnpm init
pnpm add @mysten/sui
pnpm add -D tsx typescript
```

本地解码函数：

```ts
const SIDE_BIT = 1n << 127n;
const PRICE_MASK = (1n << 63n) - 1n;
const SEQ_MASK = (1n << 64n) - 1n;

export function decodeOrderId(orderId: bigint) {
  const isBid = (orderId >> 127n) === 0n;
  const price = (orderId >> 64n) & PRICE_MASK;
  const sequence = orderId & SEQ_MASK;
  return { isBid, price, sequence };
}
```

源码对照：

- [packages/deepbook/sources/helper/utils.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/utils.move) 的 `decode_order_id`。
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move) 的 `iter_orders`。

查询方式：可用 SDK 调用 Move function 或使用项目已有 DeepBook SDK 包装。无论哪种方式，返回的 order id 都按字符串处理，再转 BigInt 解码。

核对点：

- bids 应按更优价格优先展示。
- asks 应按更低价格优先展示。
- 过滤过期订单时使用 `min_expire_timestamp`。
- 分页时保留上一页最后一个 order id，作为下一页 cursor 的参考。
