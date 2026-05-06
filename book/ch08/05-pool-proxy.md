# ch08-05 pool_proxy 如何连接 Margin 和 DeepBook Pool

[返回本章](README.md)

## 先看风险边界

Margin 下单最危险的误解，是把它看成“普通 DeepBook 下单 + 借钱”。如果应用直接拿 `BalanceManager` 调现货 Pool，就会绕过 Margin 的价格保护、池启用状态和 reduce-only 规则。

`pool_proxy` 的存在就是为了堵住这个口子：所有 Margin 交易先过 registry 和 manager，再进入 DeepBook Pool。

## 源码入口

按调用顺序看这三处：

- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)：交易代理入口。
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)：manager 里的 TradeProof 和 BalanceManager 借用。
- [packages/deepbook_margin/sources/helper/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/oracle.move)：价格保护背后的 oracle 读法。

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 关键定义

`pool_proxy` 的限价单入口把 Margin 检查放在 DeepBook 下单之前：先确认 manager 属于这个 pool，再检查 registry 是否启用，最后做 oracle 价格保护。

```move
public fun place_limit_order<BaseAsset, QuoteAsset>(
    registry: &MarginRegistry,
    margin_manager: &mut MarginManager<BaseAsset, QuoteAsset>,
    pool: &mut Pool<BaseAsset, QuoteAsset>,
    client_order_id: u64,
    order_type: u8,
    self_matching_option: u8,
    price: u64,
    quantity: u64,
    is_bid: bool,
    pay_with_deep: bool,
    expire_timestamp: u64,
    clock: &Clock,
    ctx: &TxContext,
): OrderInfo {
    assert!(margin_manager.deepbook_pool() == pool.id(), EIncorrectDeepBookPool);
    assert!(registry.pool_enabled(pool), EPoolNotEnabledForMarginTrading);

    registry.assert_price(pool.id(), price, is_bid, clock);

    let trade_proof = margin_manager.trade_proof(ctx);
    let balance_manager = margin_manager.balance_manager_trading_mut(ctx);

    pool.place_limit_order(
        balance_manager,
        &trade_proof,
        client_order_id,
        order_type,
        self_matching_option,
        price,
        quantity,
        is_bid,
        pay_with_deep,
        expire_timestamp,
        clock,
        ctx,
    )
}
```

这个代理层是 Margin 应用的关键安全边界。普通 DeepBook 下单只知道 BalanceManager 和 TradeProof；Margin 下单还必须知道账户风险、池是否启用、价格是否偏离 oracle。SDK 设计时应把这一层作为默认入口。

## 读风险控制面

常规 Margin 下单先做风控检查，再进入现货撮合：

1. `margin_manager.deepbook_pool() == pool.id()` 防止 manager 和交易对错配。
2. `registry.pool_enabled(pool)` 防止在禁用池里继续加风险。
3. 限价单直接检查用户 price；市价单先从订单簿算 effective price，再检查 oracle 偏离。
4. 从 `MarginManager` 取 `trade_proof` 和可变 `balance_manager`。
5. 调用 DeepBook Pool 的 `place_limit_order` 或 `place_market_order`。

当某个 Pool 被禁用 Margin 交易时，用户仍可能需要减少风险。`place_reduce_only_limit_order` 和 `place_reduce_only_market_order` 会读取当前债务和资产，只允许减少净负债方向的订单，否则抛出 `ENotReduceOnlyOrder`。

## 工程旁白

`pool_proxy` 的价值不只是代调用 DeepBook。它把 registry 的启用状态、oracle 价格保护、manager 的 `TradeCap` 和 reduce-only 判断放在同一条链上路径里，保证下单不会绕过 Margin 风控。

市价单需要先根据订单簿推导 effective price，再与 oracle 检查；限价单则直接检查用户给出的 price。应用侧因此要把滑点、订单簿深度和 oracle stale 风险都显示在交易预览中。

## 风控判断

- SDK 默认只暴露 Margin 下单入口；原始 DeepBook Pool 下单只能给专业或内部工具使用。
- Pool 被禁用时，界面不要简单禁用全部按钮，应保留 reduce-only 平仓或降风险路径。
- 错误提示要区分 oracle 价格保护、订单簿流动性不足、pool 未启用和 manager-pool 错配。

## 动手检查

- 这笔订单会增加债务风险，还是符合 reduce-only 的降风险方向？
- 市价单的 effective price 是否经过 `registry.assert_price` 检查？
- 应用是否在下单后调用 withdraw settled amounts 或重新读取结算余额？
