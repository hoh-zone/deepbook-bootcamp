# ch04-05 限价单进入撮合引擎的完整路径

[返回本章](README.md)

## 先看订单现场

读这一节时，不要从函数名开始。先问一笔订单走到“限价单进入撮合引擎的完整路径”这个环节时，排序、成交、挂单、取消或退款中哪一个状态会发生变化。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：`validate_inputs`、`match_maker`、`assert_execution` 和订单生命周期事件。

## 关键定义

限价单的入口签名本身就是完整参数表：

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

读这个签名时，不要只看 `price` 和 `quantity`。`self` 和 `balance_manager` 都是 `&mut`，说明一次下单会同时改变池状态和用户交易账户。`trade_proof` 是权限边界。`order_type` 和 `self_matching_option` 决定订单是否可以剩余挂单、是否必须完全成交、遇到自成交时取消哪一侧。返回值 `OrderInfo` 是本次执行结果，不能直接等同于订单簿上的 `Order`。

## 把订单走一遍

限价单入口是 `pool::place_limit_order`，参数包括 BalanceManager、TradeProof、client order id、order type、self matching option、price、quantity、side、pay_with_deep、expire timestamp、Clock。

内部 `place_order_int` 做四件事：

1. 更新 EWMA 状态并读取池的 `PoolInner`。
2. 根据 `pay_with_deep` 生成 `OrderDeepPrice`。
3. 创建 `OrderInfo` 并调用 `book.create_order`。
4. 调用 `state.process_create` 和 `vault.settle_balance_manager` 完成账户与资产结算。

`book::create_order` 的顺序非常重要：

```text
validate_inputs
set_order_id
match_against_book
assert_execution
inject_limit_order
emit_order_placed
```

如果订单完全成交、IOC 取消剩余、FOK 不满足而 abort，或者 post-only 发生 crossing abort，它不会被插入订单簿。

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

函数调用链可以按“入口参数校验 -> `OrderInfo::validate_inputs` -> `book::match_against_book` -> `OrderInfo::assert_execution`”阅读。资金路径不是在 `Book` 内直接转 coin，而是由成交结果写入账户应收应付，后续再交给 Vault 和 BalanceManager 结算。

> **Move 技巧**：撮合模块里看不到大量 `Coin<T>` 转移是正常的。高质量协议会把“计算成交结果”和“移动资产”分开，前者生成 fill 和应收应付，后者再由 `Vault` 和 `BalanceManager` 统一结算。

## 交易实现提醒

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 动手检查

- 限价单进入撮合引擎的完整路径 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
