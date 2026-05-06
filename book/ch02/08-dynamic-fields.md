# ch02-08 动态字段

[返回本章](README.md)

## 先建立手感

先不要把“动态字段”当成孤立语法点。DeepBook 里每个资产、订单和权限对象都会受 Move 类型系统约束，读这一节时要看语法如何变成资金安全边界。

## Move 对照

先用下面几处源码建立 Move 概念的落点。这里不追完整协议流程，只确认类型、ability、对象、事件和测试入口如何在真实项目中出现。

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/helper/big_vector.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/big_vector.move)
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)

读这些文件时，把语法点和真实对象放在一起看：ability、泛型、对象 ID、事件和测试入口分别承担什么约束。

## 拆开来看

动态字段适合在对象下挂载规模不固定的数据。订单簿、账户映射、注册表和余额容器都可能需要这种模式。

`balance_manager.move` 使用 `Bag` 保存不同资产余额，并定义 `BalanceKey<phantom T>` 作为资产余额的 key。源码还引入 `dynamic_field as df`，用于 referral 等对象下字段。

`registry.move` 使用动态字段和 key 管理池子、稳定币和应用授权。后续读注册表时要关注 key 类型，而不是只搜索字符串。

## 阅读补充

动态字段适合把“按类型或 key 扩展”的状态挂在对象下面。DeepBook 中授权、余额或大集合辅助结构会使用这种模式，避免在主对象 struct 中预先写死所有可能字段。

阅读动态字段时，重点找 key 类型、父对象、增删改入口和是否存在反向索引。只知道用了 dynamic field 还不够，调试时必须能从父对象 ID 和 key 还原查询路径。

## Move 判断

- 标注每个动态字段的父对象和 key 类型。
- 写查询工具时不要只拿对象字段，还要考虑动态字段分页。
- 删除授权或余额字段时检查是否同步清理相关索引或事件。

## 练习问题

- 动态字段解决 DeepBook 哪类扩展问题？
- 查询动态字段需要知道哪两个核心信息？
- 为什么动态字段会影响 Indexer 设计？
