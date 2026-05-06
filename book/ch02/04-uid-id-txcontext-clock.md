# ch02-04 UID、ID、TxContext、Clock

[返回本章](README.md)

## 先建立手感

先不要把“UID、ID、TxContext、Clock”当成孤立语法点。DeepBook 里每个资产、订单和权限对象都会受 Move 类型系统约束，读这一节时要看语法如何变成资金安全边界。

## Move 对照

先用下面几处源码建立 Move 概念的落点。这里不追完整协议流程，只确认类型、ability、对象、事件和测试入口如何在真实项目中出现。

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)
- [packages/deepbook/sources/state/history.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/history.move)

读这些文件时，把语法点和真实对象放在一起看：ability、泛型、对象 ID、事件和测试入口分别承担什么约束。

## 拆开来看

`UID` 是对象身份字段，只能在对象内部持有。`hello_move.move` 的 `UserProfile`、`AdminCap`、`MyResource` 都包含 `id: UID`。

`ID` 是对象 ID 的值形式，可复制、存储、作为事件字段。`hello_move.move` 的 `UserCreated` 事件包含 `user_id: ID`，通过 `object::id(&user)` 生成。

`TxContext` 提供交易上下文。源码中常见 `object::new(ctx)` 创建对象，`tx_context::sender(ctx)` 或 `ctx.sender()` 读取发送者，`ctx.timestamp_ms()` 读取时间戳。

`Clock` 是共享时钟对象。DeepBook 交易入口 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 `place_limit_order` 和 `place_market_order` 都接收 `clock: &Clock`，用于过期时间、订单状态和交易时间判断。

## 阅读补充

`UID` 是对象身份字段，通常只在对象 struct 内部持有；`ID` 是可复制记录的对象标识，适合写入事件或其它状态。`TxContext` 提供创建对象和发送者上下文，`Clock` 则为订单过期、历史统计和 epoch 相关逻辑提供链上时间。

阅读交易入口时，把 `ctx: &mut TxContext` 与 `clock: &Clock` 分开理解：前者服务对象创建和交易上下文，后者服务时间判断。把二者混用会导致你误判函数是否依赖当前时间。

## Move 判断

- 创建对象必须追踪 `object::new(ctx)` 的调用位置。
- 事件里通常记录 `ID` 或地址，不应暴露或移动对象的 `UID`。
- 涉及过期时间、epoch 或历史统计时查找 `Clock` 参数。

## 练习问题

- 为什么 `UID` 不能当作普通字段随意复制？
- `TxContext` 和 `Clock` 在下单入口中分别解决什么问题？
- 订单过期逻辑应该优先搜索哪个参数或模块？
