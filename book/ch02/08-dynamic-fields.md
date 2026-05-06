# ch02-08 动态字段

[返回本章](README.md)

## 本节目标

- 理解 DeepBook 如何用动态字段扩展账户、授权和集合状态。
- 能沿“动态字段”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/helper/big_vector.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/big_vector.move)
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

动态字段适合在对象下挂载规模不固定的数据。订单簿、账户映射、注册表和余额容器都可能需要这种模式。

`balance_manager.move` 使用 `Bag` 保存不同资产余额，并定义 `BalanceKey<phantom T>` 作为资产余额的 key。源码还引入 `dynamic_field as df`，用于 referral 等对象下字段。

`registry.move` 使用动态字段和 key 管理池子、稳定币和应用授权。后续读注册表时要关注 key 类型，而不是只搜索字符串。

## 阅读补充

动态字段适合把“按类型或 key 扩展”的状态挂在对象下面。DeepBook 中授权、余额或大集合辅助结构会使用这种模式，避免在主对象 struct 中预先写死所有可能字段。

阅读动态字段时，重点找 key 类型、父对象、增删改入口和是否存在反向索引。只知道用了 dynamic field 还不够，调试时必须能从父对象 ID 和 key 还原查询路径。

## 开发要点

- 标注每个动态字段的父对象和 key 类型。
- 写查询工具时不要只拿对象字段，还要考虑动态字段分页。
- 删除授权或余额字段时检查是否同步清理相关索引或事件。

## 检查问题

- 动态字段解决 DeepBook 哪类扩展问题？
- 查询动态字段需要知道哪两个核心信息？
- 为什么动态字段会影响 Indexer 设计？
