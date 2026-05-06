# ch01-07 全局心智模型

[返回本章](README.md)

## 先看问题

这一节不急着进源码。先用“全局心智模型”回答一个更基础的问题：如果要构建真实交易应用，哪些概念必须先讲清楚，哪些细节可以等到后面再拆。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)
- [packages/deepbook/sources/state/state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/state.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)
- [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 建立直觉

一次 DeepBookV3 spot 交易可以按下面路径理解：

```text
wallet -> PTB -> Pool<Base, Quote>
              -> BalanceManager + TradeProof
              -> Book match/place
              -> State account/history/governance
              -> Vault settlement
              -> event::emit
              -> Indexer -> Server/API -> frontend
```

`Pool` 在 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)，负责公开交易入口。`BalanceManager` 在 [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)，负责用户资金和交易证明。`Book` 在 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)，负责订单簿读写和撮合。`Vault` 在 [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)，负责最终资产结算。

Indexer 不应该从前端猜订单状态，而应消费事件。订单相关事件定义在 `book/order_info.move` 和 `book/order.move`，余额事件定义在 `balance_manager.move`。

## 阅读补充

全局路径可以简化为：钱包构造交易，传入 `Pool` 与 `BalanceManager` 授权；`Pool` 调用 `Book` 完成撮合；`State` 更新账户、费用和历史；`Vault` 做净额资产结算；事件被 Indexer 消费成链下查询模型。每一步都对应不同源码目录，不要用一个“成交成功”概括。

阅读时建议画两条线：控制流从 `pool.move` 进入撮合和状态处理，资金流从 `BalanceManager` 与 `Vault` 的差额结算展开。事件流则独立标注，因为它服务于查询和审计，不参与交易原子性本身。

## 落地判断

- 画架构图时同时标注控制流、资金流和事件流。
- 定位 bug 时先判断是链上执行失败、资金不足、事件索引延迟还是前端读模型错误。
- 不要把 Indexer 数据当作交易成功的先决条件，交易成功由链上执行决定。

## 读完以后问自己

- 一次下单的控制流和资金流分别经过哪些模块？
- 为什么事件流需要单独画出来？
- 前端看到订单状态延迟时，首先该怀疑链上还是 Indexer？
