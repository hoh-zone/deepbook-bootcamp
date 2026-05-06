# ch01-05 为什么 Sui 对象模型适合订单簿

[返回本章](README.md)

## 先看问题

这一节不急着进源码。先用“为什么 Sui 对象模型适合订单簿”回答一个更基础的问题：如果要构建真实交易应用，哪些概念必须先讲清楚，哪些细节可以等到后面再拆。

## 本节坐标

本节先把源码范围缩到最少。读的时候只抓产品边界、对象名称、函数入口和事件线索；后面进入交易路径时，再回到这些入口核对细节。

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/helper/big_vector.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/big_vector.move)

这一组入口用来校准产品边界：先看它们暴露哪些对象和动作，再判断本节概念最终会落到哪条交易或数据路径。

## 建立直觉

订单簿交易的难点是高并发：不同交易对、不同账户、不同订单簿层级最好能并行执行。Sui 的对象模型把状态显式放到对象里，交易声明要读写哪些 owned object、shared object 和 immutable object，执行层可以据此判断并行边界。

DeepBookV3 把交易对封装成 `Pool<BaseAsset, QuoteAsset>`，定义在 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)。`Pool` 是 `has key` 的链上对象，内部持有 `Versioned`，真实字段在 `PoolInner`：`book`、`state`、`vault`、`deep_price` 和版本集合。

实践上，应用构造交易前要先确定会触碰哪些对象：交易对池子、用户 `BalanceManager`、`Clock`、可能的 coin 或 cap。对象选择错误会导致 dev inspect 或执行时 abort。

## 阅读补充

DeepBook 的关键拆分是：池子本身是共享对象，用户交易账户由 `BalanceManager` 表示，权限通过 cap/proof 传入。这样交易必须串行访问同一个池的撮合状态，但不同用户的余额授权和链下查询可以有更清晰的对象边界。

阅读对象模型时，重点看函数签名中的 `&mut Pool`、`&mut BalanceManager`、`&Clock` 和 capability 参数。签名往往比正文注释更直接地告诉你哪些状态会被写、哪些状态只读、哪些权限必须由调用者提供。

## 落地判断

- 分析并发时先列出 mutable shared object，而不是只看函数名。
- 把 UID/ID、owned/shared/capability 的区别写进交易构造说明。
- 订单簿数据结构可以复杂，但入口对象借用关系必须先画清楚。

## 读完以后问自己

- 为什么同一个池的下单需要访问 shared `Pool`？
- `BalanceManager` 的拆分给用户授权带来什么好处？
- 函数签名里哪些参数最能体现并行边界？
