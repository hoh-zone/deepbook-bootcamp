# s04 撤单

目标：演示单笔撤单、严格批量撤单和容错批量撤单。

函数：

- `pool.cancel_order`：订单必须存在且属于当前 manager。
- `pool.cancel_orders`：任一订单失败则整笔交易失败。
- `pool.cancel_live_order`：无效订单直接跳过。
- `pool.cancel_live_orders`：批量跳过无效订单。
- `pool.cancel_all_orders`：按账户 open orders 清理。

运行方式：

```bash
pnpm install
pnpm tsx cancel-order.ts --order-id 123
pnpm tsx cancel-live-orders.ts --order-ids 123,456,789
```

资金路径：

撤单后 `state.process_cancel` 计算未成交数量退款，`vault.settle_balance_manager` 把 base/quote/DEEP 退回 `BalanceManager`，并发出订单取消事件。
