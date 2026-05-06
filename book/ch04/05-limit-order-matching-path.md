# ch04-05 限价单进入撮合引擎的完整路径

[返回本章](README.md)

## 本节目标

- 明确 限价单进入撮合引擎的完整路径 在 `Pool -> Book -> OrderInfo/Order` 撮合链中的职责。
- 能指出本节涉及的订单字段、排序字段、事件或退款字段来自哪个 Move 模块。
- 能用一笔具体订单解释价格优先、时间优先、成交、挂单、取消或查询结果如何产生。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：`validate_inputs`、`match_maker`、`assert_execution` 和订单生命周期事件。

## 正文

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

补充阅读：阅读 限价单进入撮合引擎的完整路径 时，先从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的入口确认交易类型，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消路径。taker 的即时执行状态保存在 `OrderInfo`，maker 的可挂单状态保存在 `Order`，这两个对象不要混在一起看。

函数调用链可以按“入口参数校验 -> `OrderInfo::validate_inputs` -> `book::match_against_book` -> `OrderInfo::assert_execution`”阅读。资金路径不是在 `Book` 内直接转 coin，而是由成交结果写入账户应收应付，后续再交给 Vault 和 BalanceManager 结算。

> **Move 技巧**：撮合模块里看不到大量 `Coin<T>` 转移是正常的。高质量协议会把“计算成交结果”和“移动资产”分开，前者生成 fill 和应收应付，后者再由 `Vault` 和 `BalanceManager` 统一结算。

## 开发要点

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 检查问题

- 限价单进入撮合引擎的完整路径 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
