# ch02 Sui Move 开发基础

## 本章目标

读完本章后，你应该能够：

- 读懂 DeepBook 源码中的 `module`、`struct`、`fun`、ability 和泛型。
- 解释 `key`、`store`、`copy`、`drop` 对金融协议对象的影响。
- 区分 owned object、shared object、immutable object，以及 `UID`、`ID`、`TxContext`、`Clock` 的职责。
- 理解 `Coin<T>` 与 `Balance<T>` 的转换意义。
- 找到 `event::emit` 和 abort code，并知道 Indexer 为什么依赖事件。

## 本章学习阶梯

- L1 用 `hello_move.move` 读懂 Move 基本语法。
- L2 用 `Coin<T>`、`Balance<T>`、ability 和 shared object 理解金融对象。
- L3 从 DeepBook 源码中定位事件、abort code、泛型资产和动态字段。
- L4 写一个最小 Move 练习，把语法变成可运行对象和事件。

## 源码地图

| 主题 | GitHub 源码 | 阅读重点 |
| --- | --- | --- |
| Move 包配置 | [packages/deepbook/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/Move.toml) | package 名称、edition、依赖和地址映射 |
| Move 基础示例 | [packages/deepbook/sources/hello_move.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/hello_move.move) | module、struct、ability、泛型、对象、事件 |
| 资金账户 | [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) | `Coin<T>` 入账为 `Balance<T>`，余额事件，cap 和 proof |
| 交易入口 | [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) | `Pool<BaseAsset, QuoteAsset>`、`Clock`、泛型资产 |
| 订单事件 | [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move) | 订单生命周期事件 |
| 共享状态辅助 | [packages/deepbook/sources/helper/big_vector.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/big_vector.move) | `key` 对象和动态扩展数据结构 |

## 小节目录

- [01 module、struct、fun、ability](01-move-module-struct-fun-ability.md)
- [02 ability 对金融对象的影响](02-abilities-for-financial-objects.md)
- [03 Sui 对象：owned、shared、immutable](03-sui-owned-shared-immutable-objects.md)
- [04 UID、ID、TxContext、Clock](04-uid-id-txcontext-clock.md)
- [05 泛型资产类型与 phantom](05-generic-asset-types-and-phantom.md)
- [06 Coin&lt;T&gt; 与 Balance&lt;T&gt;](06-coin-and-balance.md)
- [07 split、join、into_balance、into_coin](07-split-join-into-balance-into-coin.md)
- [08 动态字段](08-dynamic-fields.md)
- [09 event::emit 与 Indexer](09-events-and-indexer.md)
- [10 abort code](10-abort-codes.md)
- [11 Move 单元测试和 test_scenario](11-move-tests-and-test-scenario.md)
- [12 阅读 Move.toml](12-reading-move-toml.md)
- [13 本章代码](13-chapter-code.md)

## Move 高阶穿插点

- 学 Move 不要停在语法表，直接用 DeepBook 的资产、订单、cap、proof 来理解 ability 的工程后果。
- 看到 `Coin<T>` 和 `Balance<T>` 时，追踪它们在函数边界上的转换，不要把钱包资产和协议内部余额混成一个概念。
- 读 `assert!` 时把 abort code 写成“用户会看到什么错误、开发者该回到哪条交易前检查”。

## 常见错误

- 给资产、cap、proof 加上不该有的 `copy` 或 `drop`，破坏资源约束。
- 把 `ID` 当成 `UID` 使用。`ID` 是可记录的值，`UID` 是对象身份字段。
- 忘记 `TxContext`，导致无法 `object::new(ctx)` 或 `into_coin(ctx)`。
- 在 TypeScript move call 中漏写泛型资产类型。
- 只查询对象当前字段，不查询事件，导致订单历史和余额流水不完整。

## 本章检查清单

- [ ] 能解释 `Greeting has copy, drop, store` 和 `UserProfile has key` 的差异。
- [ ] 能说出 `Coin<T>` 存入 `BalanceManager` 后如何变成 `Balance<T>`。
- [ ] 能找到 `BalanceEvent` 和 `OrderPlaced` 的事件定义。
- [ ] 能根据 abort code 回到源码定位 `assert!`。
- [ ] 能读懂 [packages/deepbook/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/Move.toml)。

## 进阶练习

1. 实现一个最小共享对象计数器：`Counter has key`，包含 `increment(&mut Counter)` 和 `value(&Counter): u64`。
2. 实现一个 `Coin<T>` 存取款示例：存入时 `into_balance`，取出时 `into_coin`。
3. 给计数器增加事件 `CounterUpdated`，然后用 RPC 查询事件。
