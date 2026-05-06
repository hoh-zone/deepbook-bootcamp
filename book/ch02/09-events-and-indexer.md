# ch02-09 event::emit 与 Indexer

[返回本章](README.md)

## 本节目标

- 理解链上事件如何成为订单和余额历史的读模型来源。
- 能沿“event::emit 与 Indexer”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

链上状态告诉你“现在是什么”，事件告诉你“发生过什么”。Indexer 依赖事件重建订单、成交、余额、治理等时间序列。

`hello_move.move` 中 `create_user` 发出 `UserCreated`，`send_greeting` 发出 `GreetingSent`，`update_balance` 发出 `BalanceUpdated`。这是学习事件的最小样本。

DeepBook 真实事件更多：`balance_manager.move` 发出 `BalanceManagerEvent` 和 `BalanceEvent`；`book/order_info.move` 发出 `OrderPlaced`、`OrderFilled`、`OrderFullyFilled` 等；`book/order.move` 发出 `OrderCanceled` 和 `OrderModified`。

## 阅读补充

事件是交易执行后的链上事实记录，Indexer 把这些事实转换成查询友好的表或 API。DeepBook 的订单状态、成交、余额变化和费用解释，都不能只靠当前对象字段复原完整历史。

阅读事件时先找 `event::emit`，再看事件 struct 字段，最后看 indexer 如何消费这些字段。字段少了会影响链下可观测性，字段语义错了会让前端和报表都误读交易。

## 开发要点

- 每个用户可见状态变化都要能找到对应事件或解释为什么没有事件。
- Indexer 延迟不等于链上交易失败，前端应区分 pending indexing 和 failed execution。
- 事件字段应包含足够的对象 ID、账户 ID 和数量信息，方便回放。

## 检查问题

- 为什么当前对象字段不能替代订单历史？
- `event::emit` 到 Indexer 表之间有哪些转换步骤？
- 前端订单列表缺一笔成交时应先查 digest 还是数据库？
