# ch01-07 全局心智模型

[返回本章](README.md)

## 本节目标

- 把钱包签名、交易入口、撮合、结算和事件索引连成一张图。
- 能沿“全局心智模型”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)
- [packages/deepbook/sources/state/state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/state.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)
- [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

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

## 开发要点

- 画架构图时同时标注控制流、资金流和事件流。
- 定位 bug 时先判断是链上执行失败、资金不足、事件索引延迟还是前端读模型错误。
- 不要把 Indexer 数据当作交易成功的先决条件，交易成功由链上执行决定。

## 检查问题

- 一次下单的控制流和资金流分别经过哪些模块？
- 为什么事件流需要单独画出来？
- 前端看到订单状态延迟时，首先该怀疑链上还是 Indexer？
