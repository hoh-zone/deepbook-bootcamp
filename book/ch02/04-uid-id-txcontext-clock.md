# ch02-04 UID、ID、TxContext、Clock

[返回本章](README.md)

## 本节目标

- 理解对象身份、可记录 ID、交易上下文和链上时间的职责。
- 能沿“UID、ID、TxContext、Clock”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)
- [packages/deepbook/sources/state/history.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/history.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

`UID` 是对象身份字段，只能在对象内部持有。`hello_move.move` 的 `UserProfile`、`AdminCap`、`MyResource` 都包含 `id: UID`。

`ID` 是对象 ID 的值形式，可复制、存储、作为事件字段。`hello_move.move` 的 `UserCreated` 事件包含 `user_id: ID`，通过 `object::id(&user)` 生成。

`TxContext` 提供交易上下文。源码中常见 `object::new(ctx)` 创建对象，`tx_context::sender(ctx)` 或 `ctx.sender()` 读取发送者，`ctx.timestamp_ms()` 读取时间戳。

`Clock` 是共享时钟对象。DeepBook 交易入口 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 `place_limit_order` 和 `place_market_order` 都接收 `clock: &Clock`，用于过期时间、订单状态和交易时间判断。

## 阅读补充

`UID` 是对象身份字段，通常只在对象 struct 内部持有；`ID` 是可复制记录的对象标识，适合写入事件或其它状态。`TxContext` 提供创建对象和发送者上下文，`Clock` 则为订单过期、历史统计和 epoch 相关逻辑提供链上时间。

阅读交易入口时，把 `ctx: &mut TxContext` 与 `clock: &Clock` 分开理解：前者服务对象创建和交易上下文，后者服务时间判断。把二者混用会导致你误判函数是否依赖当前时间。

## 开发要点

- 创建对象必须追踪 `object::new(ctx)` 的调用位置。
- 事件里通常记录 `ID` 或地址，不应暴露或移动对象的 `UID`。
- 涉及过期时间、epoch 或历史统计时查找 `Clock` 参数。

## 检查问题

- 为什么 `UID` 不能当作普通字段随意复制？
- `TxContext` 和 `Clock` 在下单入口中分别解决什么问题？
- 订单过期逻辑应该优先搜索哪个参数或模块？
