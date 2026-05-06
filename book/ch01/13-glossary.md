# ch01-13 术语表

[返回本章](README.md)

## 先看问题

这一节不急着进源码。先用“术语表”回答一个更基础的问题：如果要构建真实交易应用，哪些概念必须先讲清楚，哪些细节可以等到后面再拆。

## 本节坐标

本节先把源码范围缩到最少。读的时候只抓产品边界、对象名称、函数入口和事件线索；后面进入交易路径时，再回到这些入口核对细节。

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)
- [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)

这一组入口用来校准产品边界：先看它们暴露哪些对象和动作，再判断本节概念最终会落到哪条交易或数据路径。

## 建立直觉

pool：某个 `BaseAsset/QuoteAsset` 交易对的链上池子，对应 `Pool<BaseAsset, QuoteAsset>`。

base：交易对基础资产。`SUI/USDC` 中 SUI 是 base。

quote：计价资产。`SUI/USDC` 中 USDC 是 quote。

lot：数量离散单位。订单数量必须满足最小数量和 lot size 约束。

tick：价格离散单位。限价单价格必须满足 tick size 约束。

order id：订单唯一编号，后续在 `book/order.move` 和 `book/order_info.move` 分析。

checkpoint：Sui 链上全局进度点，Indexer 用它确定事件和交易处理进度。

## 阅读补充

术语表不是翻译表，而是防止同一个词在不同层被误用。例如 account 在钱包层、`BalanceManager` 层、`state/account.move` 层含义不同；order 在前端展示、`book/order.move` 压缩状态和 `order_info.move` 生命周期对象里也不是同一个结构。

维护术语时建议写“术语 -> 源码位置 -> 在交易路径中的职责”。只写中文解释会在后续章节失去约束力，尤其是 fee、rebate、stake、settled、owed 这类容易被产品文案简化的词。

## 落地判断

- 每个核心术语都尽量附一个源码文件或后续章节入口。
- 发现同名概念跨层含义不同，要在术语表里拆分。
- 不要用业务口号替代对象、函数或事件的准确名称。

## 读完以后问自己

- `account` 在 DeepBook 文档里可能指哪几层？
- 为什么 `Order` 和 `OrderInfo` 不能混用？
- 术语表如何帮助调试交易失败？
