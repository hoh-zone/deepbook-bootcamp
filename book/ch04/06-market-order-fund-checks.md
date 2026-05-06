# ch04-06 市价单和吃单资金检查

[返回本章](README.md)

## 先看订单现场

交易界面里的“市价买入”看起来像一个简单按钮：按现在盘口立刻成交。但在 DeepBook 里，它不是一种绕过价格系统的特殊订单。更准确地说，它是一笔被推到极端价格的限价单，再用 IOC 限制剩余数量。

这个设计很重要。协议保证订单不会留下意外挂单，但它不会替应用承担滑点保护。交易终端、做市机器人或聚合器如果不先估算输出，用户仍然可能在很薄的订单簿上吃到很差的价格。

## 源码入口

这一节只需要抓住三处：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：看 `place_market_order` 如何改写 price 和 order type。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：看订单进入 `match_against_book` 后如何吃掉对手盘。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：看输入校验、成交累积和执行策略检查。

## 关键定义

市价单入口本身很短，它把价格改成极端值，并固定使用 IOC。真正的撮合与资金结算仍然走同一条内部路径。

```move
public fun place_market_order<BaseAsset, QuoteAsset>(
    self: &mut Pool<BaseAsset, QuoteAsset>,
    balance_manager: &mut BalanceManager,
    trade_proof: &TradeProof,
    client_order_id: u64,
    self_matching_option: u8,
    quantity: u64,
    is_bid: bool,
    pay_with_deep: bool,
    clock: &Clock,
    ctx: &TxContext,
): OrderInfo {
    self.place_order_int(
        balance_manager,
        trade_proof,
        client_order_id,
        constants::immediate_or_cancel(),
        self_matching_option,
        if (is_bid) constants::max_price() else constants::min_price(),
        quantity,
        is_bid,
        pay_with_deep,
        clock.timestamp_ms(),
        clock,
        true,
        ctx,
    )
}
```

这段代码说明一个重要设计：DeepBook 没有“无限滑点”的市价单类型。它用 bid 最大价或 ask 最小价把订单推入限价撮合路径，然后用 IOC 阻止剩余数量挂单。因此应用必须自己做 quote、深度检查和最小输出保护。

## 把订单走一遍

`pool::place_market_order` 没有另写一套市价撮合引擎。它把参数整理好后直接进入 `place_order_int`：

- bid 市价单使用 `constants::max_price()`。
- ask 市价单使用 `constants::min_price()`。
- order type 固定为 `immediate_or_cancel()`。
- `market_order = true`，并把 expire timestamp 设为当前时间。

这几个参数合在一起，得到的行为才像用户理解中的“市价单”：尽可能按当前订单簿成交，剩余部分不进入 book。

`order_info::validate_inputs` 对 market order 会跳过普通限价价格校验，但仍然禁止 post-only。资金检查不在 `Book` 里发生。`Book` 只产生成交；买方 owed quote 和 fee、卖方 owed base 和 fee 会进入 `OrderInfo` 和 account/vault 结算路径。如果 BalanceManager 余额不足，最后仍会在 proof 取款或结算阶段 abort。

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

生产应用不能只依赖 max/min price。下单前应先用 `book::get_quantity_out` 或 SDK quote 能力估算成交量、手续费和最差可接受输出；提交后再用事件或订单查询确认实际成交。

## 交易实现提醒

- 报价预估不是体验优化，而是市价单的安全前置条件。
- 前端展示“预计成交价”时，要同时展示深度不足和 fee 资产，尤其是 `pay_with_deep = true` 的情况。
- 事件监听要按 maker/taker 方向解释 base、quote，不能只看 `is_bid` 一个字段。

## 动手检查

- 用一个“买入 10 SUI”的例子推演：bid 市价单最终传入的 price 是什么？
- 如果订单簿只够成交 6 SUI，剩余 4 SUI 会进入 book 吗？
- 如果 dry run 失败，先排查滑点、余额、DEEP fee，还是 self matching option？
