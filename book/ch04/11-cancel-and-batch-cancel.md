# ch04-11 取消单和批量取消路径

[返回本章](README.md)

## 先看订单现场

读这一节时，不要从函数名开始。先问一笔订单走到“取消单和批量取消路径”这个环节时，排序、成交、挂单、取消或退款中哪一个状态会发生变化。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：`Order` 字段、`get_order_id`、`generate_fill`、`locked_balance` 和取消退款。

## 关键定义

严格撤单入口会先从 `Book` 移除订单，再检查订单归属。批量严格撤单只是循环调用单笔撤单，因此任何一笔失败都会让整笔交易回滚。

```move
public fun cancel_order<BaseAsset, QuoteAsset>(
    self: &mut Pool<BaseAsset, QuoteAsset>,
    balance_manager: &mut BalanceManager,
    trade_proof: &TradeProof,
    order_id: u128,
    clock: &Clock,
    ctx: &TxContext,
) {
    let self = self.load_inner_mut();
    let mut order = self.book.cancel_order(order_id);
    assert!(order.balance_manager_id() == balance_manager.id(), EInvalidOrderBalanceManager);
    let (settled, owed) = self
        .state
        .process_cancel(&mut order, balance_manager.id(), self.pool_id, ctx);
    self.vault.settle_balance_manager(settled, owed, balance_manager, trade_proof);

    order.emit_order_canceled(self.pool_id, ctx.sender(), clock.timestamp_ms());
}

public fun cancel_orders<BaseAsset, QuoteAsset>(
    self: &mut Pool<BaseAsset, QuoteAsset>,
    balance_manager: &mut BalanceManager,
    trade_proof: &TradeProof,
    order_ids: vector<u128>,
    clock: &Clock,
    ctx: &TxContext,
) {
    let mut i = 0;
    let num_orders = order_ids.length();
    while (i < num_orders) {
        let order_id = order_ids[i];
        self.cancel_order(balance_manager, trade_proof, order_id, clock, ctx);
        i = i + 1;
    }
}
```

读这段代码要注意“状态顺序”和“交易原子性”是两件事。函数体里先从 `Book` cancel，再做 owner 校验；但如果校验失败，Move 交易整体 abort，前面的状态写入不会落链。批量取消也一样，适合“全部必须成功”的管理操作；交易前端更常用 live/no-op 版本来处理已成交或已取消的订单。

## 把订单走一遍

取消入口在 `pool.move`：

- `cancel_order`：必须成功取消指定订单。
- `cancel_orders`：逐个调用 `cancel_order`，任一失败会导致整笔交易失败。
- `cancel_live_order`：如果订单不在该 BalanceManager 的 open orders 中则 no-op。
- `cancel_live_orders`：批量 no-op 版本。
- `cancel_all_orders`：从账户 open orders 取全部订单并取消。

核心链路：

```text
pool::cancel_order
  -> book::cancel_order
  -> assert order.balance_manager_id == balance_manager.id
  -> state::process_cancel
    -> order.set_canceled
    -> order.calculate_cancel_refund
    -> account.remove_order
    -> account.settle
  -> vault::settle_balance_manager
  -> order.emit_order_canceled
```

修改订单 `modify_order` 只能降低数量。它调用 `book::modify_order`，再通过 `state::process_modify` 退回减少部分对应的锁定资产和 maker fee。

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

事件阅读时要把 `OrderPlaced`、`OrderFilled`、`OrderFullyFilled`、`OrderCanceled` 和 `OrderExpired` 串成生命周期。取消和过期还会触发锁定资产释放，退款由 `Order` 计算后进入结算路径。

## 交易实现提醒

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 动手检查

- 取消单和批量取消路径 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
