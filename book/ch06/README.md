# ch06 DeepBookV3 Spot 交易开发

## 本章目标

本章从应用开发角度实现 DeepBookV3 Spot 交易：发现池子，展示 base/quote 和订单簿，创建或复用 `BalanceManager`，构造限价单、市价单、撤单 PTB，并处理成交事件、订单刷新、dry run、Gas 和常见错误。

## 本章学习阶梯

- L1 先列出 Spot 应用最小功能：池子、订单簿、余额、下单、撤单。
- L2 用 SDK/PTB 完成查询、充值、限价单、市价单和 dry run。
- L3 把每个交易调用反向映射到 `pool.move` 入口。
- L4 做出 CLI 或 Web 交易面板，并处理错误和刷新。

## 关键定义卡片

Spot 应用开发最先要读的是下单签名，而不是 SDK 封装名：

```move
public fun place_limit_order<BaseAsset, QuoteAsset>(
    self: &mut Pool<BaseAsset, QuoteAsset>,
    balance_manager: &mut BalanceManager,
    trade_proof: &TradeProof,
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
): OrderInfo
```

这就是应用侧的对象清单：`Pool`、`BalanceManager`、`TradeProof`、`Clock`、base/quote 类型参数，以及整数化后的价格和数量。前端表单里的 decimal、side、order type、过期时间和费用选项，最后都要落到这个签名上。dry run 失败时，也应按这个签名反查：对象是否同网，manager 是否有余额，proof 是否有效，price/quantity 是否符合 tick 和 lot。

## 源码地图

- `packages/deepbook/sources/pool.move`：Spot 交易对外入口。
- `packages/deepbook/sources/order_query.move`：分页读取订单簿。
- `packages/deepbook/sources/book/order_info.move`：订单生命周期、事件和错误码。
- `packages/deepbook/sources/book/order.move`：订单字段、锁定余额和取消退款。
- `packages/deepbook/sources/book/book.move`：撮合、插入、取消订单。
- `packages/deepbook/sources/state/state.move`：订单创建、取消、成交后的账户状态更新。

## 小节目录

- [01 Spot 应用的最小功能集](01-spot-minimum-feature-set.md)
- [02 交易对发现、池子选择和显示](02-pool-discovery-and-display.md)
- [03 精度、tick size、lot size 和最小量](03-precision-tick-lot-min-size.md)
- [04 钱包连接和交易签名](04-wallet-connection-signing.md)
- [05 创建 BalanceManager 和首次充值](05-balance-manager-and-first-deposit.md)
- [06 限价买单和限价卖单 PTB](06-limit-order-ptb.md)
- [07 市价买入和市价卖出 PTB](07-market-order-ptb.md)
- [08 订单状态刷新和深度展示](08-order-status-and-depth.md)
- [09 成交历史、个人成交和未成交订单](09-fills-history-open-orders.md)
- [10 撤单、批量撤单和失败重试](10-cancel-batch-cancel-retry.md)
- [11 交易前模拟、Gas 和错误提示](11-dry-run-gas-errors.md)
- [12 做市机器人数据和交易循环](12-market-maker-loop.md)
- [13 最小命令行交易终端](13-cli-trading-terminal.md)
- [14 最小 Web 交易面板](14-web-trading-panel.md)

## 本章代码

- `book/ch06/code/s01-pool-list-cli/`：列出池子和参数。
- `book/ch06/code/s02-place-limit-order/`：限价买单/卖单 PTB。
- `book/ch06/code/s03-place-market-order/`：市价单 PTB 和 dry run。
- `book/ch06/code/s04-cancel-order/`：单笔与批量撤单。
- `book/ch06/code/s05-mini-terminal/`：最小 CLI 交易终端设计。

## Move 高阶穿插点

- 写现货交易应用时，把每个 SDK 调用反向映射到 `pool.move` 的 public 函数。
- 构造 PTB 前先列对象清单：Pool、BalanceManager、Clock、Registry、Coin、type arguments，一个都不要靠猜。
- dry run 失败不是终点，要把错误归类到参数、权限、余额、版本、撮合策略或对象网络不一致。

## 常见错误

- 把市价买入数量理解为 quote 数量；源码中仍是 base quantity。
- 没有把 decimal 输入转换成 tick/lot 对齐的整数。
- 市价单未设置滑点保护或未 dry run。
- 期待所有成交订单都有 `OrderPlaced`；完全吃单不会入簿。
- 撤单重试使用严格批量入口，导致一个无效订单使整批失败。

## 本章检查清单

- 能写出限价单和市价单的 PTB 参数。
- 能按 `OrderFilled` 字段重建个人成交。
- 能使用 `order_query::iter_orders` 分页读取订单簿。
- 能区分 `cancel_order` 与 `cancel_live_order`。
- 能把 Move abort code 转成前端提示。

## 进阶练习

1. 实现一个本地订单簿聚合器，把 `iter_orders` 返回的订单按价格聚合。
2. 写一个市价单 dry run 工具，输出预估成交均价、手续费和最差可接受输出。
3. 给做市机器人加库存上限，超过上限自动停止单侧报价。
