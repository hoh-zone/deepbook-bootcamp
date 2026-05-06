# s01-orderbook-simulator

目标：用 TypeScript 实现最小订单簿模拟器，对齐 `book::match_against_book` 的价格优先行为。

建议依赖：

```bash
pnpm init
pnpm add -D tsx typescript
pnpm tsx index.ts
```

核心数据结构：

```ts
type Side = 'bid' | 'ask';

type Order = {
  id: bigint;
  side: Side;
  price: bigint;
  quantity: bigint;
  filled: bigint;
  balanceManagerId: string;
  expireTimestamp: bigint;
};
```

模拟规则：

- bid taker 匹配 asks，按价格从低到高。
- ask taker 匹配 bids，按价格从高到低。
- 只有 `bid.price >= ask.price` 才能成交。
- 成交 base quantity 是 taker remaining 与 maker remaining 的较小值。
- maker 完全成交后从订单簿删除。

与源码对应：

- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 的 `match_against_book`。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move) 的 `match_maker`。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move) 的 `generate_fill`。

扩展任务：加入 `post_only`、`immediate_or_cancel`、`fill_or_kill` 三种订单类型，并输出 taker 最终状态。
